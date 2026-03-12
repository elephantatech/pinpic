# PinPic — Product Requirements Document

## Overview

**Product Name:** PinPic
**Platform:** Meta Quest (Quest 2, Quest 3, Quest 3S; targeting Meta Horizon Store)
**Version:** 1.1
**Date:** 2026-03-11
**Engine:** Unity 6 (URP) with Meta XR All-in-One SDK, MR Utility Kit (MRUK), Interaction SDK, Passthrough Camera API

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

- Use **MRUK + Scene API** for robust flat-surface detection (tables, walls, floors, canvases)
- User selects a detected surface or manually defines a plane
- Anchor via **OVRSpatialAnchor** with 6DoF stability; persists across sessions
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
- OAuth tokens stored in OS secure storage
- No image data sent to external servers beyond chosen cloud providers
- Camera feed (PCA) processed on-device only; no frames leave the headset
- GDPR/CCPA-compliant data handling for store release

### Accessibility
- Hand tracking and controller input supported for all interactions
- UI elements minimum 48dp touch targets
- High-contrast UI mode option
- Voice commands for key actions (optional, v1.1+)

---

## Technical Architecture

### Engine & SDK
- **Engine:** Unity 6 (Universal Render Pipeline)
- **Meta SDKs:** Meta XR All-in-One SDK, MR Utility Kit (MRUK), Interaction SDK, Passthrough API, Spatial Anchors API
- **Language:** C#

### Key Components

```
PinPic/
├── Assets/
│   ├── Scripts/
│   │   ├── ImageImport/          # Local & cloud import logic
│   │   ├── SurfaceDetection/     # MRUK + Scene API integration, plane detection
│   │   ├── ImageAnchoring/       # OVRSpatialAnchor management, image placement
│   │   ├── SessionManager/       # Save/load, auto-save, session library
│   │   ├── CloudIntegration/     # Google Drive & Flickr OAuth + API
│   │   ├── MRUKIntegration/      # Scene understanding utilities
│   │   └── UI/                   # Panels, sliders, menus, onboarding
│   ├── Prefabs/
│   ├── Materials/
│   ├── Shaders/                  # Custom opacity/blend shaders
│   └── Resources/
├── Packages/
└── ProjectSettings/
```

### Cloud Integration
- Google Drive: Google Identity Services (OAuth 2.0) -> Drive API v3
- Flickr: OAuth 1.0a -> Flickr REST API
- HTTP via UnityWebRequest

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
| **Phase 1 — Prototype** | MRUK surface detection, local image import, opacity control, OVRSpatialAnchor anchoring | Week 1–4 |
| **Phase 2 — Persistence & Polish** | Session save/load, project explorer, image manipulation polish | Week 5–6 |
| **Phase 3 — Cloud & UX** | Google Drive & Flickr integration, onboarding tutorial, UI polish | Week 7–10 |
| **Phase 4 — Store Prep** | Performance optimization, VRC compliance, store assets, beta testing | Week 11–14 |

Official Meta samples (Spatial Anchors, Passthrough, MRUK) accelerate Phase 1–2 significantly.

---

## MVP v0.1 — Minimum Viable Product

The MVP focuses on the core loop: **import a local image → anchor it to a physical surface → adjust opacity → physically draw/trace on the real surface → save and resume the reference setup**.

The app is a **MR reference overlay tool** (digital lightbox). All drawing happens physically — no digital stroke tracking, no brush tools, no digital canvas. The user draws with real pens, markers, or brushes on the real surface while viewing the semi-transparent reference image through the headset.

### v0.1 Features

| # | Feature | Description |
|---|---|---|
| 1 | **Unity Project Setup** | Unity 6 (URP) project with Meta XR All-in-One SDK, MRUK, Interaction SDK, Passthrough API configured for Quest 2 & Quest 3S |
| 2 | **Passthrough Setup** | Enable and configure MR passthrough so user sees real environment with digital image overlays |
| 3 | **Local Image Import** | Browse and select images from Quest device storage (Downloads, Pictures). Support JPEG, PNG. Display in a file picker UI panel |
| 4 | **Surface Detection** | Use MRUK + Scene API to detect flat surfaces. Highlight detected planes for user selection |
| 5 | **Image Anchoring** | Pin selected image to chosen surface via OVRSpatialAnchor. 6DoF stability as user moves head |
| 6 | **Opacity Control** | Slider UI to adjust anchored image opacity (0–100%, default 40%) so user can see through to the physical surface for tracing |
| 7 | **Image Manipulation** | Grab-to-reposition, pinch-to-scale, rotation, lock in place, horizontal/vertical flip |
| 8 | **Project Explorer** | Save/load sessions: persist anchoring data (anchor pose, image reference, opacity, scale/rotation/flip/lock state) and project metadata. Session list UI with thumbnails. Delete/duplicate projects |

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
| **Sprint 1 — Foundation** (Week 1–2) | 2 weeks | Unity project setup, Meta SDK integration, passthrough config, surface detection with MRUK |
| **Sprint 2 — Image Pinning** (Week 3–4) | 2 weeks | Local file picker, image loading, anchoring to surface, opacity shader, image manipulation (move/scale/rotate/lock/flip) |
| **Sprint 3 — Persistence & Polish** (Week 5–6) | 2 weeks | Project explorer UI, session save/load with spatial anchors, thumbnail generation, basic QA on Quest 2 & Quest 3S |

---

## v0.2 — Art Progress & Sharing

Build on v0.1 by adding the ability to document physical art progress and share it.

### v0.2 Features

| # | Feature | Description |
|---|---|---|
| 1 | **Progress Photo Capture** | Capture snapshots of the physical artwork via passthrough camera (PCA). User triggers capture manually (hand gesture or controller button). Photos saved to session folder with timestamp. Captures the real surface without the reference overlay |
| 2 | **Photo Gallery** | View captured progress photos within the session in chronological order. Delete individual photos. Full-screen preview |
| 3 | **Progress Animation** | Auto-generate a time-lapse animation from captured progress photos. Configurable frame duration (0.5s–3s per photo). Export as MP4 video. Preview animation in-app before exporting |
| 4 | **Share Photos** | Share individual progress photos to social media or other apps via Android share intent (system share sheet). Save to device gallery |
| 5 | **Share Animation** | Share generated MP4 animation to social media via Android share intent. Save to device storage |
| 6 | **Cloud Image Import** | Google Drive and Flickr integration for importing reference images (OAuth 2.0) |
| 7 | **Multi-Reference Images** | Support multiple pinned reference images per session with per-image visibility toggle |

### v0.2 Technical Notes
- Photo capture uses **Passthrough Camera API (PCA)** to grab frames from headset cameras
- Reference overlay should be hidden/removed from the captured frame so the photo shows only the physical art
- Animation generation: stitch PNGs into MP4 using Unity's built-in video encoding or a lightweight encoder
- Share via Android `Intent.ACTION_SEND` / `Intent.ACTION_SEND_MULTIPLE`
- Quest 2 photos will be grayscale; Quest 3/3S will be color

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Quest 2 grayscale passthrough limits reference clarity | Lower contrast reference on Quest 2 | Optimize shader for grayscale; market primarily to Quest 3/3S users |
| Spatial anchor drift over long sessions | Reference image misalignment | Periodic re-anchoring prompts; manual nudge adjustment UI |
| Meta store rejection | Launch delay | Follow VRC guidelines from day 1; submit early for concept review |
| OAuth token management on headset | Security, UX friction | Use system browser for OAuth flows; secure token storage |
| Thermal throttling during long sessions | Frame drops, degraded tracking | Monitor thermal state; warn user; reduce passthrough processing if needed |

---

## Resolved Questions (from v1.0)

1. **Non-flat surfaces:** Defer to v2.0
2. **Pricing model:** One-time $9.99 (no ongoing server costs to justify subscription)
3. **Additional cloud sources:** Defer to v1.1+ (iCloud, Dropbox, OneDrive)

---

## Testing Plan

### Unit Tests (NUnit in Unity)
- Anchor save/load persistence
- Session serialization/deserialization
- Image loading and downsampling logic

### Integration / Playmode Tests
- XR Interaction Simulator (hand/controller input simulation)
- Full flow: import -> anchor -> adjust opacity -> save -> reload session
- Cloud OAuth flow (mocked endpoints)

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

**Tooling:** GitHub Actions + Game.ci (free tier)

```yaml
# .github/workflows/build.yml
name: Build & Test
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: dotnet-format check
        run: dotnet format --verify-no-changes

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: game-ci/unity-test-runner@v4
        with:
          unityVersion: '6000.0.0f1'
          testMode: 'editmode'

  build:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: game-ci/unity-builder@v4
        with:
          unityVersion: '6000.0.0f1'
          targetPlatform: Android
          androidAppBundle: false
      - uses: actions/upload-artifact@v4
        with:
          name: PinPic-APK
          path: build/
```

**Stages:** Lint -> Unit/Integration tests -> Build APK -> Upload artifact
**Deployment:** Sideload to test devices; upload to Meta Release Channel (Alpha/Beta) for staged rollout.

---

## Linting, Formatting & Code Quality

- **.editorconfig** at project root (Unity-recommended C# settings)
- **Roslyn Analyzers + StyleCop** for style enforcement (via NuGet/Unity package)
- **dotnet-format** in pre-commit hooks + CI enforcement (fail build on warnings)
- **Naming conventions:** PascalCase for classes/methods, camelCase for private fields (standard Unity + Meta samples)
- Code reviews mandatory on all PRs

---

## Git Workflow

- `main` branch protected; no direct pushes
- Feature branches (`feature/`, `fix/`, `chore/`) with PRs and required reviews
- Automated tests must pass before merge
- Semantic commit messages (e.g., `feat: add stroke smoothing`, `fix: anchor drift on resume`)

---

## Dependencies & Versioning

- All Meta SDKs managed via Unity Package Manager (manifest.json)
- Pin exact SDK versions to avoid breaking changes
- Unity 6 LTS pinned to specific patch version
- Third-party packages documented in `Packages/manifest.json`

---

## Analytics & Telemetry

- Unity Analytics for usage tracking:
  - Session duration, completion rate
  - Feature usage (import source, manipulation actions, opacity preferences)
  - Drop-off points in onboarding
  - Crash reporting
- Meta's optional crash reporting integration
- All analytics opt-in with clear user consent

---

## Localization

- Unity Localization package
- v1.0: English
- v1.1: Spanish, French, German
- All UI strings externalized from day 1

---

## Post-Launch Roadmap

- **v1.1:** Logitech MX Ink stylus support, additional cloud sources (Dropbox, OneDrive), localization expansion, voice commands
- **v2.0:** Curved/irregular surface support, multi-reference images, AR phone companion app

---

## Success Metrics

- **Anchor stability:** < 2mm drift over 30-minute session
- **Onboarding completion:** >= 85% of users pin their first image in < 5 minutes
- **Session stability:** < 1% crash rate per session
- **Store rating:** >= 4.0 stars within first 90 days
- **Downloads:** 1,000+ in first 90 days post-launch

---

## Key Resources & References

- Meta Spatial Anchors & MRUK docs/samples
- Passthrough API samples
- Meta Quest developer documentation & VRC guidelines
