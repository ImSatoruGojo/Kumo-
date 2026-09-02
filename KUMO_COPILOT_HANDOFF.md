# KUMO — COMPLETE COPILOT DEVELOPMENT HANDOFF

## Mission

Build Kumo as a real native Android application, not a website. The app must ultimately compile into an installable Android APK.

Primary test device:
- Vivo Y12, model 1904
- Android 11
- Low-end hardware, so performance and RAM usage matter

The final app must also be designed to work on many other Android devices.

## Required technology

- Kotlin
- Android Studio / Gradle
- Jetpack Compose
- AndroidX Media3 ExoPlayer
- Room database
- OkHttp or Retrofit
- Coil for image loading
- Coroutines and Flow
- Material 3 only where useful; do not make the UI look like a generic Material demo

Use current stable Android libraries and APIs. Do not use deprecated ExoPlayer v2 package names; use AndroidX Media3.

## First rule

Do not fake functionality or create placeholder implementations while claiming a feature works.

Build in phases. The project must compile after every major milestone.

---

# APP CONCEPT

Kumo is a unified Android media application for:

- Anime
- Movies
- TV Shows
- Cartoons
- Manga

The defining feature is a unified catalog.

The user should NOT see ten copies of the same show because different providers return it.

Example:

Provider A: One Piece
Provider B: ONE PIECE
Provider C: One-Piece

Kumo should merge these into one title card:

One Piece

Providers should remain internal. The user normally searches, opens a title, chooses an episode, and watches.

If a provider fails, Kumo can use another compatible provider internally when legally and technically appropriate.

Do not include torrent features, watch rooms, synchronized viewing rooms, or multiplayer viewing.

---

# USER INTERFACE

## Initial visual style

- Black or very dark background
- White Kumo text/logo
- Clean, lightweight interface
- Streaming-service-style browsing
- Avoid excessive animations
- Optimized for lower-end devices

Later themes can be added in Settings.

## Beta label

Show a small notice near the top center:

Kumo is still in beta — expect some bugs

Use a yellow/orange badge. It must not block normal navigation.

## Main sections

- Home
- Search
- Library
- Settings

Media categories on Home and Search:

- Anime
- Movies
- TV Shows
- Cartoons
- Manga

---

# HOME SCREEN

Include sections such as:

- Continue Watching
- Recently Watched
- Popular
- Trending
- Recommended
- Recently Added

Use poster cards and lazy image loading.

Do not load every image or provider result at startup.

---

# SMART SEARCH

Search should use approximately 250 ms debounce.

Results should be grouped by media type.

For a search such as "one":

Anime:
- One Piece
- One Punch Man

Movies:
- relevant movie matches

TV Shows:
- relevant show matches

Cartoons:
- relevant cartoon matches

Manga:
- relevant manga matches

Search ranking priority:

1. Exact title
2. Starts with query
3. Contains query
4. Partial word match
5. Similar spelling
6. Related titles

If the user searches "One Piece", show the exact title prominently, followed by genuinely similar results.

Do not duplicate the same title merely because multiple providers return it.

---

# UNIFIED CATALOG AND DUPLICATE MERGING

Every media item should have a stable Kumo content ID independent of providers.

Example:

kumo:title:one-piece

Compare:

- normalized title
- alternative titles
- aliases
- punctuation
- capitalization
- release year when available
- media type
- season metadata where available

Keep provider IDs internally.

Suggested architecture:

UI
  -> Unified Catalog
      -> Metadata Normalizer
      -> Duplicate Merger
      -> Provider Registry
          -> Provider Adapters

Do not merge unrelated titles simply because their names are somewhat similar. Use confidence scoring and metadata.

---

# PROVIDER AND REPOSITORY ARCHITECTURE

Kumo should have a modular provider architecture.

Provider capabilities may include:

- search
- metadata
- episodes
- streams
- quality options
- audio tracks
- subtitles
- manga chapters

Settings should eventually contain an Extensions and Providers section with:

- Add Repository URL
- Refresh Repositories
- Check Compatibility
- Installed Providers
- Enable / Disable Provider
- Remove Provider
- Check for Updates
- Load Recommended Compatible Repositories

IMPORTANT:
Do not assume extensions from other applications can automatically run inside Kumo.

Repository formats must be identified and validated. Build adapters only for formats that can be supported safely and legally.

Do not blindly download and execute arbitrary internet code.

Repository safety statuses can include:

- Trusted
- Verified
- Unverified
- Suspicious
- Incompatible

Checks may include manifest validation, known hashes/signatures where available, suspicious permissions, malformed files, and compatibility.

Never claim a scanner can guarantee something is virus-free.

---

# TITLE DETAILS PAGE

When opening a title, show:

- poster
- background artwork when available
- title
- alternative title where useful
- description
- metadata
- Play / Resume button
- Add to Watchlist
- season selector
- episode list

Episode layouts:

1. Separate seasons
2. All episodes in one continuous list

Episode rows should support:

- episode number
- title when available
- watched status
- progress

---

# BUILT-IN PLAYER

Use AndroidX Media3 ExoPlayer.

Suggested core classes:

core/player/
- PlayerManager
- KumoMediaSourceFactory
- TrackManager
- IntroSkipManager
- CacheManager
- DownloadManager
- PlaybackProgressManager
- PlayerErrorRecovery

## Supported source types

Design for:

- Progressive MP4 / MKV / WebM where supported
- HLS
- DASH
- compatible subtitle tracks
- compatible alternate audio tracks

Do not promise support for every arbitrary container or stream.

## Player features

- fullscreen playback
- play / pause
- seek
- resume position
- quality selection when variants are available
- Auto quality
- audio track selection
- subtitle track selection
- playback speed
- double-tap seeking
- screen lock
- autoplay next episode
- intro skip
- episode information at upper left
- built-in player

## Quality options

Only display qualities actually provided by the stream.

Possible UI:

- Auto
- Maximum Available
- 1080p
- 720p
- 480p
- 360p
- Data Saver

Users can configure a default preference.

## Audio and language

Allow available audio tracks and subtitles.

Preferences should be separate for:

- Anime
- Movies
- TV Shows

When media provides multiple languages, automatically prefer the user's selected language but allow manual changes.

Do not invent dub/sub tracks that do not exist.

## Playback speed

Support:

- 0.5x
- 0.75x
- 1x
- 1.25x
- 1.5x
- 2x

Use Media3 PlaybackParameters with pitch preservation where supported.

## Double tap seek

Left side: seek backward
Right side: seek forward

Configurable durations:

- 5 seconds
- 10 seconds
- 15 seconds
- 30 seconds

## Screen lock

Lock must prevent accidental seeking and accidental episode changes while retaining a deliberate unlock method.

## Autoplay

Optional autoplay of next episode.

## Intro skip

Data model:

contentId
introStartMs
introEndMs

Use reliable metadata markers initially.

When playback enters an intro range:

- allow Skip Intro action
- optionally auto-skip if enabled

Skip by seeking to introEndMs.

Store preferences and markers locally.

---

# SUBTITLES

Support:

- embedded tracks
- WebVTT where supported
- TTML where supported
- compatible external subtitle files

Provide:

- subtitle language selection
- enable / disable
- styling settings later
- subtitle timing offset as an advanced feature

Subtitles should remain synchronized with playback speed using player position.

---

# CACHING AND OFFLINE

Use Media3-compatible caching.

Goals:

- configurable cache size
- LRU eviction
- cache useful segments during streaming
- do not consume unlimited storage

Offline downloads are a later milestone.

When implementing them:

- use Media3 download infrastructure
- clearly display storage requirements
- allow deletion
- preserve progress
- respect applicable rights and stream restrictions

Do not bypass DRM or access restrictions.

---

# PLAYER RESILIENCE

Listen for playback errors.

Possible safe recovery behavior:

1. Retry when appropriate
2. Use cached data if available
3. Attempt another compatible quality
4. Attempt another internally registered compatible source if the application legitimately has one

Track:

- buffering
- dropped frames
- estimated bandwidth
- startup time
- errors

Keep telemetry local by default unless a user-visible privacy policy and opt-in system are implemented.

---

# WATCH PROGRESS

Store locally using Room:

- content ID
- episode ID
- position
- duration
- last watched timestamp
- watched state

Home screen should use this for Continue Watching.

---

# MANGA

Later support:

- manga search
- unified title merging
- manga details
- chapter lists
- reader
- reading progress
- continue reading

Keep manga provider handling separate from video provider handling.

---

# SETTINGS

## General

- Theme
- App language
- Cache limit
- Image loading behavior

## Content

- preferred anime audio language
- preferred movie audio language
- preferred TV audio language
- subtitle preference

## Player

- default quality
- Data Saver
- playback speed
- double-tap duration
- autoplay
- auto-skip intro
- episode layout

## Performance

- cache size
- image quality
- background loading behavior
- low-data mode

## Providers

- repository management
- provider enable/disable
- compatibility
- safety status
- update controls

---

# PROFILES AND FRIEND FEATURES — ON HOLD

Do not prioritize these until the core app works.

Future ideas:

- username
- local gallery profile image
- personal watchlist
- public/shared watchlist
- limited friend activity

Private content must not automatically appear in shared watchlists or public activity.

Do not implement a backend account system in the first milestone.

---

# PERFORMANCE REQUIREMENTS

The Vivo Y12 is a low-end test target.

Prioritize:

- low memory use
- lazy lists
- lazy image loading
- image caching
- limited background work
- no unnecessary provider-wide requests
- hardware decoding when available
- avoid expensive animations
- avoid large uncompressed resources

Do not optimize only for one phone. Test against multiple Android API levels when possible.

---

# DATABASE MODELS

Suggested entities:

ContentEntity
- contentId
- title
- normalizedTitle
- type
- description
- posterUrl
- backdropUrl
- year

ProviderMappingEntity
- contentId
- providerId
- providerContentId
- confidence

EpisodeProgressEntity
- contentId
- episodeId
- positionMs
- durationMs
- watched
- updatedAt

IntroMarkerEntity
- contentId
- episodeId
- startMs
- endMs

UserSettingsEntity or DataStore preferences

---

# SUGGESTED PROJECT STRUCTURE

Kumo/
  app/
  core/
    common/
    model/
    network/
    database/
    player/
    provider/
  feature/
    home/
    search/
    details/
    player/
    library/
    settings/
    manga/
  providers/
    api/
    registry/
    adapters/
  ui/
    theme/
    components/

Use clear Gradle modules only if they genuinely improve build time and maintainability. Do not over-engineer the first version.

---

# DEVELOPMENT ROADMAP

## Milestone 1 — Compiling Android app

Create a genuine Android Studio project that builds.

Implement:

- app icon / Kumo branding
- splash screen
- navigation
- Home
- Search
- Library
- Settings
- placeholder local demo data

Generate a debug APK successfully.

## Milestone 2 — Core data architecture

Implement:

- models
- Room
- repositories
- unified catalog logic
- duplicate detection

## Milestone 3 — Details and episodes

Implement:

- title pages
- seasons
- episode lists
- watch progress

## Milestone 4 — Player

Implement Media3:

- progressive playback
- HLS/DASH support
- tracks
- subtitles
- speed
- double-tap seeking
- screen lock
- resume
- intro markers

## Milestone 5 — Provider interfaces

Implement interfaces and a registry.

Do not hardcode providers throughout UI code.

## Milestone 6 — Repository management

Implement:

- repository metadata
- compatibility detection
- update system
- safety warnings

Only support extension formats that have a legitimate and technically compatible implementation.

## Milestone 7 — Manga

Add manga provider interfaces and reader.

## Milestone 8 — Downloads and advanced features

Implement:

- controlled caching
- supported offline downloads
- advanced subtitle settings
- performance tuning

## Milestone 9 — On-hold features

Profiles and optional friend features.

---

# BUILD REQUIREMENTS

The project must include:

- Gradle wrapper
- settings.gradle.kts
- root build.gradle.kts
- app/build.gradle.kts
- AndroidManifest.xml
- required source sets
- GitHub Actions workflow for debug APK builds

The GitHub workflow should:

1. Check out source
2. Set up JDK
3. Set up Android/Gradle environment
4. Run the Gradle debug build
5. Upload the resulting APK as an artifact

Do not claim the workflow works until it has successfully completed.

---

# IMPORTANT IMPLEMENTATION RULES FOR COPILOT

1. Inspect existing code before replacing it.
2. Preserve working code.
3. Make small, compilable changes.
4. Run or validate builds after significant changes.
5. Fix compilation errors before adding unrelated features.
6. Never claim a feature is complete if only its UI exists.
7. Keep provider logic separate from UI.
8. Do not expose multiple duplicate provider results to normal users.
9. Do not blindly execute downloaded extension code.
10. Do not add torrent, watch-room, or synchronized group-watching systems.
11. Optimize for low-end Android devices without making the app device-specific.
12. Use modern AndroidX Media3 APIs.
13. Respect content rights, DRM, and source restrictions.

---

# IMMEDIATE TASK FOR COPILOT

Start by inspecting the current repository.

If it is not already a valid Android Gradle project, transform it into one.

The first concrete target is:

A native Android Kumo app that successfully compiles into a debug APK and launches on Android 11.

The first launchable version should contain:

- black/dark Kumo interface
- Kumo logo/title
- beta notice
- Home categories
- Search UI with category grouping and local demo data
- Library
- Settings
- title details screen
- season and episode UI

Do not wait to design every future feature before making the first APK.

After the first APK compiles successfully, continue feature-by-feature following the roadmap above.
