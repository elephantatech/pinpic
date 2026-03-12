# PinPic — Product Requirements Document

## Overview

**Product Name:** PinPic
**Platform:** Meta Quest (Quest 2, Quest 3, Quest 3S; targeting Meta Horizon Store)
**Version:** 1.1
**Date:** 2026-03-11
**Engine:** Unity 6 (URP) with Meta XR All-in-One SDK, MR Utility Kit (MRUK), Interaction SDK, Passthrough Camera API

PinPic is a mixed-reality (MR) application that lets users import reference images, anchor them to physical surfaces, reduce their opacity, and trace/draw/color on the real surface beneath. The app tracks the user's physical drawing in real time — via hand tracking or controller — creating a digital copy of the artwork alongside the physical one.

---

## Problem Statement

Artists, hobbyists, and creators who want to trace or reference images while drawing on physical surfaces currently rely on projectors, lightboxes, or printouts. These methods are expensive, inflexible, or low quality. A mixed-reality approach lets users pin any digital image onto any surface at any size and opacity — then draw on the real surface while the app captures a digital version of their work.

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
- As a user, I want the app to track my drawing digitally so I have a clean digital copy without scanning.
- As a user, I want to save and resume sessions so I can work on a drawing across multiple sittings.
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
- **Lock:** Lock image in place to prevent accidental moves while drawing
- **Flip/Mirror:** Horizontal and vertical flip

### 4. Physical Drawing Tracking

- Track hand (v2.2+) or controller using Quest hand tracking and controller APIs
- Detect contact or near-contact with anchored surface via proximity raycast + physics
- Record stroke path, pressure approximation (speed/distance heuristic), and timestamp
- Real-time stroke smoothing (Catmull-Rom / Bezier interpolation)
- Render digital overlay of tracked strokes on the surface in real time

**Drawing Tools:**
- Brush size slider (1–50 px)
- Eraser mode (erase digital strokes only)
- Undo / Redo (last 50 strokes)
- Color picker with recently-used palette

### 5. Color Detection & Tracking

- Use **Passthrough Camera API (PCA)** to sample color at the fingertip/pen tip region (maps 3D world position to camera pixel via MRUK)
- Accurate color sampling on Quest 3/3S (color passthrough)
- Fallback to manual color picker on Quest 2 (grayscale passthrough)
- Recently-used color palette + manual color override/correction
- Optional: auto-detect dominant colors from reference image for quick palette generation

### 6. Digital Canvas & Export

- Maintain a 2D digital canvas that mirrors the physical drawing
- Vector-based stroke storage (paths + pressure data) for scalable output
- Layers: reference image layer(s) (bottom), tracked drawing layer (top)
- Export options:
  - Drawing only (PNG with transparency)
  - Drawing + reference composite (PNG/JPEG)
  - SVG (vector export)
  - Time-lapse video (MP4)
  - Session replay data (proprietary format for in-app playback)
- Save to device storage or share to Google Drive

### 7. Session Management

- Auto-save sessions every 60 seconds
- Resume previous sessions with spatial anchors (same physical location)
- Session library with thumbnail previews
- Delete / duplicate sessions
- Session metadata: creation date, duration, stroke count, reference images used

### 8. UX & Onboarding

- First-launch tutorial: spatial hand-guided walkthrough covering import, anchor, draw, export
- In-app contextual help panels (dismissible)
- Calibration wizard for surface alignment on first anchor
- Optional voice commands for key actions (v1.1+)

---

## Non-Functional Requirements

### Performance
- Maintain 72 Hz refresh on Quest 2, 90 Hz on Quest 3/3S
- Passthrough latency must not exceed platform baseline
- Image anchoring drift < 2mm over a 30-minute session
- Stroke tracking latency < 20ms from physical motion to digital render
- Sustained thermal target < 40C; display battery impact warning at < 20%

### Compatibility
- **Quest 2:** Full feature set; color detection fallback to manual picker (grayscale passthrough)
- **Quest 3 / Quest 3S:** Full feature set with color passthrough for accurate color detection
### Privacy & Data
- OAuth tokens stored in OS secure storage
- No image data sent to external servers beyond chosen cloud providers
- Camera feed (PCA) processed on-device only; no frames leave the headset
- GDPR/CCPA-compliant data handling for store release

### Accessibility
- Hand tracking and controller input supported for all interactions
- UI elements minimum 48dp touch targets
- High-contrast UI mode option
- Colorblind-friendly palette indicators
- Voice commands for key actions (optional, v1.1+)

---

## Technical Architecture

### Engine & SDK
- **Engine:** Unity 6 (Universal Render Pipeline)
- **Meta SDKs:** Meta XR All-in-One SDK, MR Utility Kit (MRUK), Interaction SDK, Passthrough API, Passthrough Camera API (PCA), Spatial Anchors API
- **Language:** C#

### Key Components

```
PinPic/
├── Assets/
│   ├── Scripts/
│   │   ├── ImageImport/          # Local & cloud import logic
│   │   ├── SurfaceDetection/     # MRUK + Scene API integration, plane detection
│   │   ├── ImageAnchoring/       # OVRSpatialAnchor management, image placement
│   │   ├── DrawingTracker/       # Hand/controller tracking, stroke recording
│   │   │   ├── StrokeRecorder.cs
│   │   │   ├── StrokeSmoother.cs       # Catmull-Rom / Bezier smoothing
│   │   │   └── StrokeDataModel.cs      # Serializable vector paths + pressure
│   │   ├── ColorSampler/         # Passthrough color sampling
│   │   │   └── PassthroughColorSampler.cs  # PCA integration
│   │   ├── DigitalCanvas/        # 2D canvas rendering, layer compositing, export
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

### Stroke Data Model

```csharp
[Serializable]
public class StrokeData
{
    public List<Vector3> Points;       // World-space positions
    public List<float> Pressures;      // 0.0 - 1.0 per point
    public List<float> Timestamps;     // Seconds since session start
    public Color StrokeColor;
    public float BrushSize;            // 1 - 50 px
    public InputSource Source;         // Hand, Controller
}
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
| **Phase 2 — Drawing Tracking** | Hand/controller tracking, stroke recording + smoothing, digital canvas rendering | Week 5–8 |
| **Phase 3 — Color & Polish** | PCA color detection, export (PNG/SVG/MP4), session management | Week 9–12 |
| **Phase 4 — Cloud & UX** | Google Drive & Flickr integration, onboarding tutorial, UI polish | Week 13–16 |
| **Phase 5 — Store Prep** | Performance optimization, VRC compliance, store assets, beta testing | Week 17–20 |

Official Meta samples (Spatial Anchors, Passthrough, MRUK) accelerate Phase 1–2 significantly.

---

## Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Quest 2 grayscale passthrough limits color detection | Degraded color accuracy on Quest 2 | Fallback to manual color picker; market primarily to Quest 3/3S users |
| Spatial anchor drift over long sessions | Drawing misalignment | Periodic re-anchoring prompts; manual nudge adjustment UI |
| Hand tracking occlusion near surface | Lost strokes | Support controller as reliable alternative; surface proximity threshold tuning |
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
- Stroke recording and smoothing logic
- Color sampling accuracy (mock PCA)
- Anchor save/load persistence
- Session serialization/deserialization
- Stroke data model validation

### Integration / Playmode Tests
- XR Interaction Simulator (hand/controller input simulation)
- Full flow: import -> anchor -> draw -> export
- Multi-reference image management
- Cloud OAuth flow (mocked endpoints)

### Device Testing (Quest 2 & Quest 3S)
- Accuracy: >= 90% stroke fidelity (internal grid test)
- Performance: 72/90 Hz sustained, < 20ms stroke latency
- Thermal profiling: < 40C sustained over 30-minute session
- Battery impact measurement
- Edge cases: hand occlusion near surface, anchor drift, low-light environments, multi-reference scenes
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
  - Feature usage (import source, hand vs. controller, export format)
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

- **v1.1:** Logitech MX Ink stylus support (pressure-sensitive drawing, haptics, OpenXR Interaction Profile), additional cloud sources (Dropbox, OneDrive), localization expansion, voice commands
- **v2.0:** Curved/irregular surface support, collaborative drawing, AR phone companion app

---

## Success Metrics

- **Tracking accuracy:** >= 90% stroke path fidelity vs. physical drawing (measured in internal testing)
- **Onboarding completion:** >= 85% of users complete first drawing in < 10 minutes
- **Session stability:** < 1% crash rate per session
- **Store rating:** >= 4.0 stars within first 90 days
- **Downloads:** 1,000+ in first 90 days post-launch

---

## Key Resources & References

- Meta Spatial Anchors & MRUK docs/samples
- Passthrough Multiple-Feature Sample (brushes, color styles)
- Unity-PassthroughCameraApiSamples (color sampling)
- Meta Quest developer documentation & VRC guidelines
