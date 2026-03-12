# PinPic — Product Requirements Document

## Overview

**Product Name:** PinPic
**Platform:** Meta Quest (Quest 2, Quest 3, Quest 3S; targeting Meta Horizon Store)
**Version:** 1.2
**Date:** 2026-03-11
**Stack:** Native Android (Kotlin + C/C++) with OpenXR, Vulkan rendering, Meta OpenXR extensions

PinPic is a mixed-reality (MR) reference overlay application. Users import images, anchor them to physical surfaces with adjustable opacity, and then physically trace or draw on the real surface using real pens, markers, or brushes. The app acts as a digital lightbox — replacing projectors, lightboxes, and printouts with a precise, flexible MR overlay.

---

## Problem Statement

Artists, hobbyists, and creators who want to trace or reference images while drawing on physical surfaces currently rely on projectors, lightboxes, or printouts. These methods are expensive, inflexible, or low quality. A mixed-reality approach lets users pin any digital image onto any surface at any size and opacity — then draw on the real surface with their own physical tools while viewing the overlay through the headset.

---

## Target Users

- Artists and illustrators who trace or use reference images
- Hobbyists learning to draw or paint
- Crafters transferring patterns onto surfaces (wood, fabric, walls)
- Educators teaching art fundamentals

---

## User Stories

- As an artist, I want to pin a reference image to my canvas so I can trace proportions accurately without a projector.
- As a hobbyist, I want to adjust the opacity of my reference so I can see my physical surface clearly while still following the guide.
- As a crafter, I want to resize and position a pattern on my workpiece so I can transfer it precisely.
- As a user, I want to save and resume my reference setup so I can continue a drawing across multiple sittings.
- As a user, I want to import reference images from Google Drive so I can access my cloud library without transferring files manually.

---

## Core Features

### 1. Image Import

| Source | Details |
|---|---|
| **Local Storage** | Import from Quest device storage (Downloads, Pictures, Screenshots) |
| **Google Drive** | OAuth 2.0 integration; browse and select images from user's Drive |
| **Flickr** | OAuth integration; search public images or access user's own photos |

**Supported formats:** JPEG, PNG, WebP, BMP
**Max resolution:** 8192 x 8192 px (downsampled for performance if needed)
**Multi-reference:** Support multiple pinned reference images per session with per-image visibility toggle

### 2. Surface Detection & Image Anchoring

- Use **OpenXR Scene Understanding extension** (`XR_META_scene`) for flat-surface detection (tables, walls, floors, canvases)
- User selects a detected surface or manually defines a plane
- Anchor via **OpenXR Spatial Anchors** (`XR_FB_spatial_entity`) with 6DoF stability; persists across sessions
- Image persists in position through head movement and minor surface occlusion
- Manual calibration nudge UI for fine-tuning anchor placement
- Periodic drift correction prompt for sessions > 15 minutes

### 3. Image Manipulation

- **Resize:** Pinch-to-scale or exact dimension input (cm/inches)
- **Reposition:** Grab and drag to move on the surface plane
- **Rotate:** Two-hand twist or rotation dial
- **Opacity control:** Slider from 0% (invisible) to 100% (fully opaque), default 40%
- **Lock:** Lock image in place to prevent accidental moves while tracing
- **Flip/Mirror:** Horizontal and vertical flip

### 4. Session Management

- Auto-save sessions every 60 seconds
- Resume previous sessions with spatial anchors (same physical location)
- Session library with thumbnail previews
- Delete / duplicate sessions
- Session metadata: creation date, last modified, reference image used, display settings

### 5. UX & Onboarding

- First-launch tutorial: spatial hand-guided walkthrough covering import, anchor, adjust, and draw
- In-app contextual help panels (dismissible)
- Calibration wizard for surface alignment on first anchor
- Optional voice commands for key actions (v1.1+)

---

## Non-Functional Requirements

### Performance
- Maintain 72 Hz refresh on Quest 2, 90 Hz on Quest 3/3S
- Passthrough latency must not exceed platform baseline
- Image anchoring drift < 2mm over a 30-minute session
- Sustained thermal target < 40C; display battery impact warning at < 20%

### Compatibility
- **Quest 2:** Full feature set (grayscale passthrough)
- **Quest 3 / Quest 3S:** Full feature set (color passthrough)

### Privacy & Data
- OAuth tokens stored in Android Keystore
- No image data sent to external servers beyond chosen cloud providers
- Camera feed processed on-device only; no frames leave the headset
- GDPR/CCPA-compliant data handling for store release

### Accessibility
- Hand tracking and controller input supported for all interactions
- UI elements minimum 48dp touch targets
- High-contrast UI mode option
- Voice commands for key actions (optional, v1.1+)

---

## Technical Architecture

### Stack
- **Platform:** Native Android (API level 32+, Quest OS)
- **Languages:** Kotlin (application logic, UI, session management) + C/C++ (OpenXR runtime, Vulkan rendering, performance-critical paths)
- **XR Runtime:** OpenXR 1.0 via Meta OpenXR Mobile SDK
- **Rendering:** Vulkan 1.1 (Quest GPU — Adreno 650 on Quest 2, Adreno 740 on Quest 3/3S)
- **Build:** Gradle + CMake (for native C/C++ libs)

### Cross-Platform Portability Strategy

OpenXR is the industry-standard cross-platform XR API. The architecture is designed so the core rendering and interaction logic is portable, with vendor-specific extensions abstracted behind interfaces.

**Portable layers (no changes needed):**
- OpenXR session lifecycle, frame loop, input actions
- Vulkan rendering pipeline, shaders, texture management
- Session data model (JSON), image loading, business logic

**Abstracted behind interfaces (swap per platform):**
- Passthrough: `XR_FB_passthrough` (Meta) → `XR_HTC_passthrough` (HTC) / platform equivalent
- Scene Understanding: `XR_META_scene` → `XR_MSFT_scene_understanding` (WMR/Qualcomm)
- Spatial Anchors: `XR_FB_spatial_entity` → `XR_MSFT_spatial_anchor` (WMR) / `XR_ML_spatial_anchor` (Magic Leap)
- Hand Tracking: `XR_EXT_hand_tracking` (already cross-platform)

**Target future platforms:**
- Qualcomm Snapdragon XR (Android-based — same Kotlin + C++ stack, swap OpenXR loader)
- HTC Vive Focus / XR Elite (Android-based — same stack)
- Windows Mixed Reality / SteamVR (port Android layer to desktop, keep C++/Vulkan core)
- Apple Vision Pro (requires significant platform layer rewrite — Swift + ARKit/CompositorServices)

### OpenXR Extensions Used

| Extension | Purpose |
|---|---|
| `XR_FB_passthrough` | MR passthrough layer rendering |
| `XR_FB_passthrough_camera_access` | Camera frame access for v0.2 photo capture |
| `XR_META_scene` | Scene understanding — detect surfaces (walls, floors, tables) |
| `XR_FB_spatial_entity` | Create and manage spatial anchors |
| `XR_FB_spatial_entity_storage` | Persist anchors across sessions (local storage) |
| `XR_FB_spatial_entity_query` | Query previously saved anchors on session resume |
| `XR_FB_hand_tracking_mesh` / `XR_EXT_hand_tracking` | Hand tracking input for manipulation gestures |
| `XR_FB_swapchain_update_state` | Efficient swapchain management for overlay rendering |
| `XR_FB_composition_layer_alpha_blend` | Alpha-blended image overlay on passthrough |

### Key Components

```
pinpic/
├── app/
│   ├── src/main/
│   │   ├── java/com/elephantatech/pinpic/
│   │   │   ├── PinPicActivity.kt            # Main XR activity (lifecycle, OpenXR session)
│   │   │   ├── ui/
│   │   │   │   ├── FilePicker.kt             # Local file browser UI
│   │   │   │   ├── SessionExplorer.kt        # Project explorer / session list
│   │   │   │   ├── ImageControls.kt          # Opacity slider, manipulation controls
│   │   │   │   └── PanelRenderer.kt          # 3D UI panel rendering in XR space
│   │   │   ├── session/
│   │   │   │   ├── SessionManager.kt         # Save/load/auto-save logic
│   │   │   │   ├── SessionData.kt            # Data classes for session persistence
│   │   │   │   └── AnchorManager.kt          # Spatial anchor create/save/query/restore
│   │   │   ├── image/
│   │   │   │   ├── ImageLoader.kt            # Load, decode, downsample images
│   │   │   │   └── ImageTransform.kt         # Scale, rotate, flip, position state
│   │   │   └── cloud/                        # v0.2: Google Drive & Flickr OAuth + API
│   │   │       ├── GoogleDriveClient.kt
│   │   │       └── FlickrClient.kt
│   │   ├── cpp/
│   │   │   ├── CMakeLists.txt
│   │   │   ├── pinpic_openxr.cpp             # OpenXR lifecycle, session, frame loop
│   │   │   ├── passthrough.cpp               # Passthrough layer setup
│   │   │   ├── scene_understanding.cpp       # Surface detection via XR_META_scene
│   │   │   ├── spatial_anchors.cpp           # Anchor CRUD via XR_FB_spatial_entity
│   │   │   ├── hand_tracking.cpp             # Hand tracking input processing
│   │   │   ├── renderer/
│   │   │   │   ├── vulkan_renderer.cpp       # Vulkan init, swapchain, frame submission
│   │   │   │   ├── image_quad.cpp            # Textured quad rendering with alpha blend
│   │   │   │   ├── ui_renderer.cpp           # UI panel rendering in 3D space
│   │   │   │   └── shaders/
│   │   │   │       ├── image_overlay.vert    # Vertex shader for anchored image
│   │   │   │       ├── image_overlay.frag    # Fragment shader with opacity uniform
│   │   │   │       ├── ui_panel.vert
│   │   │   │       └── ui_panel.frag
│   │   │   └── jni_bridge.cpp                # JNI interface between Kotlin and C++
│   │   ├── res/
│   │   │   ├── layout/                       # Minimal — most UI rendered in XR
│   │   │   ├── values/
│   │   │   └── drawable/
│   │   └── AndroidManifest.xml
│   ├── build.gradle.kts
│   └── proguard-rules.pro
├── gradle/
├── build.gradle.kts                          # Root build file
├── settings.gradle.kts
├── CMakeLists.txt                            # Top-level CMake for native libs
└── PRD.md
```

### Rendering Pipeline
1. OpenXR frame loop (`xrWaitFrame` → `xrBeginFrame` → `xrEndFrame`)
2. Passthrough layer rendered as base composition layer (`XR_FB_passthrough`)
3. Image quad rendered via Vulkan into a projection layer, alpha-blended over passthrough
4. UI panels rendered as additional quad layers in XR space
5. Opacity controlled via fragment shader uniform — no texture re-upload needed

### Session Data Format
```json
{
  "version": 1,
  "id": "uuid-v4",
  "name": "My Drawing",
  "createdAt": "2026-03-11T10:00:00Z",
  "modifiedAt": "2026-03-11T12:30:00Z",
  "anchor": {
    "uuid": "spatial-anchor-uuid",
    "position": [0.0, 0.75, -0.5],
    "rotation": [0.0, 0.0, 0.0, 1.0]
  },
  "image": {
    "filename": "reference.png",
    "originalPath": "/storage/emulated/0/Pictures/myart.png"
  },
  "display": {
    "opacity": 0.4,
    "scale": [1.0, 1.0],
    "rotation": 0.0,
    "flipH": false,
    "flipV": false,
    "locked": false
  }
}
```

### Cloud Integration
- Google Drive: Google Identity Services (OAuth 2.0) → Drive API v3
- Flickr: OAuth 1.0a → Flickr REST API
- HTTP via OkHttp / Retrofit

---

## Meta Horizon Store Requirements

### Submission Checklist (per VRC guidelines, Dec 2025)
- [ ] Pass Meta Quest Technical Review (VRC — Virtual Reality Check)
- [ ] Maintain minimum 72 fps with no dropped frames during review session
- [ ] Implement proper guardian/boundary system respect
- [ ] Include first-time user tutorial / onboarding flow
- [ ] Privacy policy URL hosted and linked
- [ ] Age rating classification completed
- [ ] Store listing assets: icon (512x512), hero image, 3+ screenshots, video trailer
- [ ] Support Meta's required entitlement check for paid apps
- [ ] Localization: English minimum; Spanish, French, German recommended for broader reach

### Store Positioning
- **Category:** Creativity / Productivity
- **Price model:** $9.99 one-time purchase (IAP for additional cloud sources in future updates)
- **Rating target:** General audiences

---

## Milestones

| Phase | Scope | Target |
|---|---|---|
| **Phase 1 — Prototype** | OpenXR + Vulkan setup, passthrough, surface detection, local image import, opacity control, spatial anchor anchoring | Week 1–4 |
| **Phase 2 — Persistence & Polish** | Session save/load, project explorer, image manipulation polish | Week 5–6 |
| **Phase 3 — Cloud & UX** | Google Drive & Flickr integration, onboarding tutorial, UI polish | Week 7–10 |
| **Phase 4 — Store Prep** | Performance optimization, VRC compliance, store assets, beta testing | Week 11–14 |

Meta OpenXR Mobile SDK samples (spatial anchors, passthrough, scene understanding) accelerate Phase 1 significantly.

---

## MVP v0.1 — Minimum Viable Product

The MVP focuses on the core loop: **import a local image → anchor it to a physical surface → adjust opacity → physically draw/trace on the real surface → save and resume the reference setup**.

The app is a **MR reference overlay tool** (digital lightbox). All drawing happens physically — no digital stroke tracking, no brush tools, no digital canvas. The user draws with real pens, markers, or brushes on the real surface while viewing the semi-transparent reference image through the headset.

### v0.1 Features

| # | Feature | Description |
|---|---|---|
| 1 | **Native Android + OpenXR Setup** | Android project (Kotlin + C/C++) with Meta OpenXR Mobile SDK, Vulkan rendering, configured for Quest 2 & Quest 3S |
| 2 | **Passthrough Setup** | Enable MR passthrough via `XR_FB_passthrough` so user sees real environment with digital image overlays |
| 3 | **Local Image Import** | Browse and select images from Quest device storage (Downloads, Pictures). Support JPEG, PNG. File picker UI rendered in XR space |
| 4 | **Surface Detection** | Use `XR_META_scene` to detect flat surfaces. Highlight detected planes for user selection |
| 5 | **Image Anchoring** | Pin selected image to chosen surface via `XR_FB_spatial_entity`. 6DoF stability as user moves head |
| 6 | **Opacity Control** | Slider UI to adjust anchored image opacity (0–100%, default 40%) via fragment shader uniform |
| 7 | **Image Manipulation** | Grab-to-reposition, pinch-to-scale, rotation, lock in place, horizontal/vertical flip |
| 8 | **Project Explorer** | Save/load sessions: persist anchoring data (anchor UUID, image reference, opacity, scale/rotation/flip/lock state) and project metadata as JSON. Session list UI with thumbnails. Delete/duplicate projects |

### v0.1 Out of Scope (Backlog)

- Art progress photo capture & animation → v0.2
- Share photos/animation to social media → v0.2
- Cloud image import (Google Drive, Flickr) → v0.2
- Multi-reference images per session → v0.2
- Onboarding tutorial → v0.3
- Analytics & telemetry → v1.0
- Localization → v1.1
- Voice commands → v1.1
- Logitech MX Ink stylus support → v1.1
- Store submission → v1.0

### v0.1 Sprint Plan

| Sprint | Duration | Scope |
|---|---|---|
| **Sprint 1 — Foundation** (Week 1–2) | 2 weeks | Android project setup, OpenXR session lifecycle, Vulkan rendering pipeline, passthrough config, surface detection with `XR_META_scene` |
| **Sprint 2 — Image Pinning** (Week 3–4) | 2 weeks | Local file picker, image loading + Vulkan texture upload, anchoring to surface with `XR_FB_spatial_entity`, opacity shader, image manipulation (move/scale/rotate/lock/flip) |
| **Sprint 3 — Persistence & Polish** (Week 5–6) | 2 weeks | Project explorer UI, session save/load with spatial anchor persistence, thumbnail generation, basic QA on Quest 2 & Quest 3S |

---

## v0.2 — Art Progress & Sharing

Build on v0.1 by adding the ability to document physical art progress and share it.

### v0.2 Features

| # | Feature | Description |
|---|---|---|
| 1 | **Progress Photo Capture** | Capture snapshots of the physical artwork via `XR_FB_passthrough_camera_access`. User triggers capture manually (hand gesture or controller button). Photos saved to session folder with timestamp. Captures the real surface without the reference overlay |
| 2 | **Photo Gallery** | View captured progress photos within the session in chronological order. Delete individual photos. Full-screen preview |
| 3 | **Progress Animation** | Auto-generate a time-lapse animation from captured progress photos. Configurable frame duration (0.5s–3s per photo). Export as MP4 video. Preview animation in-app before exporting |
| 4 | **Share Photos** | Share individual progress photos to social media or other apps via Android share intent (system share sheet). Save to device gallery |
| 5 | **Share Animation** | Share generated MP4 animation to social media via Android share intent. Save to device storage |
| 6 | **Cloud Image Import** | Google Drive and Flickr integration for importing reference images (OAuth 2.0) |
| 7 | **Multi-Reference Images** | Support multiple pinned reference images per session with per-image visibility toggle |

### v0.2 Technical Notes
- Photo capture uses `XR_FB_passthrough_camera_access` to grab camera frames
- Reference overlay hidden from captured frame so the photo shows only the physical art
- Animation generation: stitch PNGs into MP4 using Android MediaCodec + MediaMuxer (hardware-accelerated, no third-party dependency)
- Share via Android `Intent.ACTION_SEND` / `Intent.ACTION_SEND_MULTIPLE`
- Quest 2 photos will be grayscale; Quest 3/3S will be color

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Quest 2 grayscale passthrough limits reference clarity | Lower contrast reference on Quest 2 | Optimize shader for grayscale; market primarily to Quest 3/3S users |
| Spatial anchor drift over long sessions | Reference image misalignment | Periodic re-anchoring prompts; manual nudge adjustment UI |
| Native OpenXR + Vulkan complexity vs. engine approach | Longer initial development time | Use Meta OpenXR samples as starting point; modular C++ architecture |
| Meta store rejection | Launch delay | Follow VRC guidelines from day 1; submit early for concept review |
| OAuth token management on headset | Security, UX friction | Use system browser for OAuth flows; Android Keystore for token storage |
| Thermal throttling during long sessions | Frame drops, degraded tracking | Monitor thermal state; warn user; reduce passthrough processing if needed |

---

## Resolved Questions (from v1.0)

1. **Engine:** Pure native Android (Kotlin + C/C++ + OpenXR + Vulkan) — no Unity, full code ownership, portable to other OpenXR headsets
2. **Non-flat surfaces:** Defer to v2.0
3. **Pricing model:** One-time $9.99 (no ongoing server costs to justify subscription)
4. **Additional cloud sources:** Defer to v1.1+ (iCloud, Dropbox, OneDrive)

---

## Testing Plan

### Unit Tests (JUnit + Kotlin Test)
- Session serialization/deserialization (JSON)
- Image loading and downsampling logic
- Anchor data persistence
- Image transform state calculations

### Instrumented Tests (Android Instrumented / Espresso)
- Full flow: import → anchor → adjust opacity → save → reload session
- File picker permissions and file access
- Cloud OAuth flow (mocked endpoints)

### Native Tests (Google Test / Catch2)
- OpenXR extension wrapper correctness
- Vulkan resource lifecycle (texture upload, cleanup)
- Shader compilation validation

### Device Testing (Quest 2 & Quest 3S)
- Performance: 72/90 Hz sustained with anchored image overlay
- Thermal profiling: < 40C sustained over 30-minute session
- Battery impact measurement
- Anchor drift measurement over 30-minute sessions
- Edge cases: anchor drift, low-light environments, large images
- Usability: 10+ beta users (internal + Meta Horizon beta program)

### VRC Compliance
- Run official Meta Mobile test plan + VRC validator before submission
- Manual review of guardian respect, onboarding, privacy policy

### Beta & Post-Launch
- Closed beta (20 users) via Meta Horizon beta channel
- Monitor crash rate (< 1%), store rating, session completion analytics

---

## CI/CD Pipeline

**Tooling:** GitHub Actions

```yaml
# .github/workflows/build.yml
name: Build & Test
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Kotlin lint (ktlint)
        run: ./gradlew ktlintCheck
      - name: C++ lint (clang-tidy)
        run: ./gradlew clangTidyCheck

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - name: Unit tests
        run: ./gradlew testDebugUnitTest
      - name: Native tests
        run: ./gradlew runNativeTests

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'
      - name: Build APK
        run: ./gradlew assembleDebug
      - uses: actions/upload-artifact@v4
        with:
          name: PinPic-APK
          path: app/build/outputs/apk/debug/
```

**Stages:** Lint (Kotlin + C++) → Unit/Native tests → Build APK → Upload artifact
**Deployment:** Sideload to test devices via `adb install`; upload to Meta Release Channel (Alpha/Beta) for staged rollout.

---

## Linting, Formatting & Code Quality

- **Kotlin:** ktlint + detekt for style and static analysis
- **C/C++:** clang-format (LLVM style) + clang-tidy for static analysis
- **GLSL shaders:** Manual review (no standard linter)
- **.editorconfig** at project root
- Pre-commit hooks via pre-commit framework
- Code reviews mandatory on all PRs

---

## Git Workflow

- `main` branch protected; no direct pushes
- Feature branches (`feature/`, `fix/`, `chore/`) with PRs and required reviews
- Automated tests must pass before merge
- Semantic commit messages (e.g., `feat: add surface detection`, `fix: anchor drift on resume`)

---

## Dependencies & Versioning

- Meta OpenXR Mobile SDK — pin exact version
- Vulkan SDK headers — bundled with Android NDK
- OkHttp / Retrofit — for cloud API calls (v0.2+)
- Kotlin serialization — for JSON session data
- All versions pinned in `build.gradle.kts` and `CMakeLists.txt`

---

## Analytics & Telemetry

- Firebase Analytics (lightweight, works on Quest) for usage tracking:
  - Session duration, completion rate
  - Feature usage (import source, manipulation actions, opacity preferences)
  - Drop-off points in onboarding
  - Crash reporting (Firebase Crashlytics)
- Meta's optional crash reporting integration
- All analytics opt-in with clear user consent

---

## Localization

- Android string resources (`res/values/strings.xml`, `res/values-es/`, etc.)
- v1.0: English
- v1.1: Spanish, French, German
- All UI strings externalized from day 1

---

## Post-Launch Roadmap

- **v1.1:** Logitech MX Ink stylus support, additional cloud sources (Dropbox, OneDrive), localization expansion, voice commands
- **v2.0:** Port to additional XR platforms (Qualcomm XR, HTC Vive Focus), curved/irregular surface support, multi-reference images, AR phone companion app

---

## Success Metrics

- **Anchor stability:** < 2mm drift over 30-minute session
- **Onboarding completion:** >= 85% of users pin their first image in < 5 minutes
- **Session stability:** < 1% crash rate per session
- **Store rating:** >= 4.0 stars within first 90 days
- **Downloads:** 1,000+ in first 90 days post-launch

---

## Key Resources & References

- Meta OpenXR Mobile SDK documentation and samples
- Meta OpenXR extension specs (passthrough, scene, spatial entity)
- Vulkan Programming Guide / Quest GPU optimization docs
- Meta Quest developer documentation & VRC guidelines
