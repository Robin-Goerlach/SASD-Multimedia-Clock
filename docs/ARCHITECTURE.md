# SASD Multimedia Clock — Software Architecture

| Field | Value |
| --- | --- |
| Document ID | SASD-MC-ARCH-001 |
| Version | 0.1 (architecture proposal) |
| Date | 2026-11-09 |
| Status | **Proposed — review required; not an implementation claim** |
| Primary targets | iOS / iPadOS (universal app) |
| Proposed deployment target | iOS / iPadOS 16.0, pending P0 validation |
| Related documents | [Requirements / Lastenheft](requirements/Lastenheft-v0.1.md), [Technical specification / Pflichtenheft](requirements/Pflichtenheft-v0.1.md), [Research](research/AppStore-Feature-Research.md) |

> **Architecture rule:** A scheduled intent is not a guarantee that the operating system will execute it. In particular, a third-party iOS app must not promise that it can wake itself, unlock the device, and start arbitrary video or internet radio while suspended or terminated. All capability claims require measured evidence.

## 1. Purpose, scope, and decision status

This document specifies the **proposed system architecture** for SASD Multimedia Clock. It translates the German requirements documents into architectural boundaries, dependencies, interfaces, lifecycle policies, data ownership, security constraints and verification gates. It is intended to guide implementation, code review and future architecture decision records (ADRs).

**Decided product principles:** one native iPhone/iPad application; bedside, desk and multimedia profiles; support for older working devices as a first-class goal; customizable skins; local-first configuration; optional sleep assistance disabled by default; honest disclosure of platform limitations.

**Proposed technical choices, pending validation:** Swift and SwiftUI app shell, AVFoundation-backed media adapters, platform alarm/notification adapters, structured local persistence, protocol-based dependency injection, and a small set of bounded feature modules. Neither a C++20 core nor a plugin runtime is required for V1.

**Out of scope for this document:** final UI pixel specifications, a guaranteed App Store release date, a working iOS project, a general-purpose browser, DRM circumvention, unsupported background playback, and promises of Android/Windows/Linux portability in V1.

### 1.1 Architectural quality priorities

In descending order where trade-offs conflict:

1. **Correctness and truthful alarm behavior**, including permission and capability disclosure.
2. **Data integrity and recoverability** for alarms, settings, profiles and media sources.
3. **Compatibility and efficient operation** on older iPhones and iPads.
4. **Usability and accessibility** for bedside, desk and multimedia operation.
5. **Security, privacy and predictable offline behavior**.
6. **Modularity and testability** without premature abstraction.
7. **Future portability**, only when it does not jeopardize the above.

## 2. System context and trust boundaries

```text
                       ┌─────────────────────────────────────┐
                       │ iOS / iPadOS                         │
                       │ Clock · lifecycle · audio session    │
                       │ Notifications · permissions · files  │
                       └─────────────────┬───────────────────┘
                                         │ platform APIs
┌───────────────────┐        ┌───────────▼────────────────────┐
│ User              │◄──────►│ SASD Multimedia Clock          │
│ touch/VoiceOver   │  UI    │ UI · domain · adapters · store  │
└───────────────────┘        └───────┬─────────┬──────────────┘
                                    │         │
                           network  │         │ local authorized files
                                    ▼         ▼
                          ┌──────────────┐ ┌────────────────┐
                          │ Stream / API │ │ Device storage │
                          │ providers    │ │ & media access │
                          └──────────────┘ └────────────────┘
```

External services are **untrusted inputs**: stream hosts, podcast feeds, weather providers, calendar integrations and downloadable skin packages. The app must not assume that URLs remain reachable or that provider terms permit recording, caching or redistribution.

The OS owns execution eligibility, device time, notification delivery, app lifecycle, audio interruption behavior and permission decisions. The application owns *intent*, local settings, user-visible state, source validation and recoverable operations.

### 2.1 Deployment contexts

- **Bedside:** mostly stationary, often plugged in, dimmed display, minimal interaction; must avoid burn-in and excess heat.
- **Desk:** prolonged foreground display, modular dashboard, frequent interruptions and multitasking.
- **Multimedia:** explicit foreground media playback, possible external audio routes, network variability.
- **Offline/legacy device:** clock and local configuration must remain usable without network; unsupported OS versions must be clearly rejected.

## 3. Architectural style and dependency rules

Use a **modular monolith with ports and adapters** (hexagonal boundaries) and unidirectional UI state. This is one application, not microservices and not an in-app plugin execution platform.

```text
SwiftUI Screens / Widgets / Accessibility
                  │ commands, read-only state
                  ▼
Application coordinators (use cases, workflows, policies)
                  │
                  ▼
Pure domain models and rules (no UIKit, SwiftUI, AVFoundation)
                  ▲
                  │ ports / protocols
                  │
Infrastructure adapters (OS, AVFoundation, network, persistence)
```

**Dependency constraints**
- Domain models and pure scheduling rules must not import SwiftUI, UIKit, AVFoundation or concrete persistence frameworks.
- Application services depend on domain protocols, not on specific Apple APIs.
- Platform adapters implement domain-facing ports; they do not own product policy.
- UI invokes application commands and observes immutable view state; it must not schedule notifications or open network connections directly.
- Cross-feature coordination happens in application orchestration, not by feature modules importing each other arbitrarily.
- No global mutable singleton for player, alarms or settings. Use explicit app-scoped ownership and injected dependencies.
- Avoid generic frameworks until at least two real use cases justify them.

### 3.1 Ownership and concurrency

A single app-scoped **composition root** creates services and wires dependencies. UI-facing observable state is updated on the main actor. I/O and decoding happen asynchronously outside rendering paths. Use Swift structured concurrency with cancellation; isolate mutable shared state in actors or explicitly synchronized components. Do not use detached tasks as a default. A clock ticker must not be used as a scheduling authority.

## 4. Proposed module catalog

| Module | Responsibility | Explicitly does **not** own |
| --- | --- | --- |
| AppShell | startup, composition, scene lifecycle, routing | domain rules |
| ClockDomain | time formatting, zones, date/time presentation models | periodic UI rendering |
| ProfileService | active profile, per-profile settings, migration | playback engine |
| AlarmDomain | alarm definitions, recurrence, next occurrence, DST policy | OS alarm delivery |
| AlarmCoordinator | reconcile intent with available platform delivery | unsupported wake guarantees |
| RoutineEngine | policy-bounded sequences of actions | system wake/unlock |
| MediaCatalog | favorites, source descriptors, history policy | AVPlayer |
| MediaResolver | URL normalization, scheme/format validation, metadata | DRM bypass |
| PlaybackCoordinator | player lifecycle, state, audio session policy | alarm delivery |
| SkinSystem | validated manifests, theme tokens, layout contracts | arbitrary code execution |
| Dashboard | widget configuration, data snapshots, stale states | third-party permissions |
| SleepAssistance | opt-in sessions, fade/timer behavior | automatic implicit activation |
| TimerService | timers and elapsed-time policy | guaranteed execution when terminated |
| SettingsStore | versioned durable settings, backups/migrations | UI decisions |
| Diagnostics | structured, privacy-safe events and troubleshooting | covert telemetry |
| PlatformAdapters | notifications, files, audio, network, power hints | business policy |

**Module contracts are more important than folder names.** Start with Swift targets/packages only where tests or build isolation benefit. A multi-target architecture is optional, not a V1 acceptance criterion.

## 5. Critical subsystem: time, alarms and routines

### 5.1 Three separate concepts

1. **Clock:** visual representation of current wall time.
2. **Alarm:** persisted user intent to receive a wake-up alert, delivered only through available OS-supported mechanisms.
3. **Routine:** best-effort media/display actions contingent on app execution and permissions.

These concepts must not share a misleading `guaranteedExecution` flag.

### 5.2 Clock and calendar semantics

- Store absolute instants in UTC when appropriate, but persist **local wall-clock intent** for recurring alarms (e.g. 07:00 every weekday) together with the intended time zone policy.
- Use system calendar/time-zone APIs; never assume 24-hour days or fixed UTC offsets.
- Define behavior for daylight saving time **gaps** and **repeated times**; decide explicitly whether a repeated occurrence fires once or twice. Default proposal: at most once per intended local date, subject to delivery adapter support.
- Recompute next occurrences after time-zone changes, clock changes, reboot/relaunch and alarm edits.
- A timer uses a monotonic elapsed-time source while running; persisted timers need a documented recovery rule after termination/reboot.
- UI time updates may be throttled by visibility and desired granularity; no busy-loop rendering.

### 5.3 Alarm lifecycle

```text
Draft → Validated → Saved → Reconciled with OS
                             │
               ┌─────────────┼─────────────┐
               ▼             ▼             ▼
           Scheduled      Degraded      Unsupported
               │             │             │
               ▼             └────► User-visible explanation
          Delivered*
               │
          Acknowledged / Snoozed / Expired
```

`Delivered*` means **observed or reported by the platform**, not a guarantee that the user heard the sound. Store distinct states for `configured`, `permissionGranted`, `scheduledWithOS`, `deliveryObserved`, `userAcknowledged`, and `fallbackAvailable`. Avoid falsely marking an alarm as successfully delivered based only on a successful scheduling API call.

**Reconciliation algorithm:** persist validated alarm intent first; attempt adapter scheduling; record adapter result and OS identifiers; retry reconciliation on next suitable lifecycle event; show unresolved/degraded state in UI. If storage and OS scheduling disagree, **do not silently claim success**.

### 5.4 Platform capability abstraction

```swift
protocol AlarmDeliveryPort {
    func capabilities() async -> AlarmCapabilities
    func reconcile(_ alarms: [AlarmDefinition]) async -> [AlarmScheduleResult]
    func cancel(platformIdentifier: String) async throws
}
```

The interface is illustrative, not a finalized API. Implementations may use platform notification facilities and, on newer supported OS versions, dedicated alarm APIs after validation. A local notification is not semantically identical to a native Clock alarm. Do not silently fall back to media playback and call it an alarm.

### 5.5 Routine engine

Routines are explicit, ordered, cancellable steps with preconditions:

```text
Trigger (time / user action)
  → Evaluate capabilities & consent
  → Check app execution eligibility
  → Set optional audio policy
  → Resolve source
  → Start playback IF allowed
  → Report outcome or safe fallback
```

A routine must have an idempotency key, cancellation policy, per-step timeout and error classification. Duplicate triggers must not launch multiple players or reapply a volume ramp. Actions affecting device settings must only use supported public APIs; there is no general right to set system volume or screen brightness from background execution.

### 5.6 P0 alarm evidence matrix

Test at minimum: foreground; background; locked; force-quit; reboot; offline; denied notification permission; Do Not Disturb/Focus; muted device; Bluetooth route changes; daylight-saving transition; manual time change; battery saver; app update; and low storage. Record **device model, OS version, API, exact expected/observed result, timestamp, logs and limitations**. Force-quit and reboot tests must not be inferred from simulator results.

## 6. Media architecture

### 6.1 Source types and contracts

Use a typed source model rather than treating all URLs as equivalent:

```text
MediaSource
  id, displayName, origin, kind, locator,
  authenticationReference?, declaredFormat?,
  artworkReference?, policy, metadata
```

Kinds initially: `directAudioURL`, `directVideoURL`, `localFile`, `podcastFeed`, `playlistReference`; add provider-backed types only after feasibility and legal review. `origin` distinguishes user entry, bundled sample and imported source.

A website URL is not a stream. Authentication credentials must never be embedded in diagnostic logs or plain-text exported configuration.

### 6.2 Resolver pipeline

```text
User input
  → Parse & canonicalize
  → Scheme/host policy (HTTPS preferred)
  → Resolve redirects within limits
  → Inspect content type / bounded header
  → Parse supported playlist or feed safely
  → Check codec/container capability
  → Return playable descriptor OR typed failure
  → PlaybackCoordinator
```

Apply request timeouts, redirect limits, bounded downloads, bounded feed parsing and explicit error categories. Reject `file://` paths outside authorized sandbox/security-scoped access. Prevent local/private network probing by untrusted imported sources where feasible; treat user-entered LAN sources separately with explicit consent. Never fetch arbitrary embedded URLs from a skin manifest.

### 6.3 Playback coordinator

Expose a single user-facing playback state machine:

```text
idle → resolving → preparing → playing ↔ paused
                 ↘ failed       ↘ buffering
playing / paused → stopping → idle
```

Transitions must be observable, cancellable and testable. Enforce **one active primary media session** unless a deliberate mixing policy is specified. Audio session configuration is centralized, including interruptions (calls/Siri), route changes, headphones, Bluetooth, silent switch expectations, AirPlay eligibility and background-audio entitlements.

**Background playback:** only where the OS permits it, with the correct declared capability and genuine user-initiated playback. Background audio does **not** imply background video autoplay or an alarm guarantee.

### 6.4 Resilience and offline behavior

- Use bounded exponential reconnect for eligible live streams, with user-visible retry state.
- Distinguish network failure, HTTP error, unsupported codec, expired credentials, geographic restrictions and DRM.
- Persist favorites/source definitions, not unnecessary streaming content.
- Resume local media only when file permissions remain valid.
- Do not cache protected media or assume offline availability of streaming services.
- Provide a locally available alarm fallback only when technically supported by the chosen alarm delivery mechanism.

## 7. UI, profiles, dashboards and skins

### 7.1 Presentation architecture

Use SwiftUI views backed by small feature view models or observable presentation stores. Keep domain logic outside views. Route user actions through typed commands. Handle scene phases and iPad multitasking without assuming the app is always full-screen.

Profiles are **configurations** over a shared capability set:

```text
Profile
  id, name, presetKind, enabledModules,
  layoutReference, skinReference, perModuleSettings,
  displayPolicy, version
```

Switching profiles must not delete alarms, source favorites or user-created skins. Specify which state is global (alarms, media catalog) and which is profile-scoped (layout, brightness preference, visible widgets, optional sleep module).

### 7.2 Skin system: data, not executable plugins

For V1, use a **versioned declarative skin format** with strict schema validation. A skin describes design tokens, typography choices, asset references and supported layout slots; it cannot run scripts, load arbitrary native code or request permissions.

Example **illustrative**, not a frozen schema:

```json
{
  "schemaVersion": 1,
  "id": "org.sasd.skin.classic-radio",
  "name": "Classic Radio",
  "minimumRendererVersion": 1,
  "tokens": {
    "background": "#171717",
    "foreground": "#F2D3A1",
    "accent": "#F7A548"
  },
  "layout": "bedside-classic",
  "assets": []
}
```

Validation includes schema version, IDs, asset paths, byte/pixel limits, unsupported features, contrast and fallback. Asset references must be package-relative; no path traversal, remote scripts or embedded executable content. A failed skin import must leave the previous active skin intact.

The renderer owns accessible semantics and dynamic content. Skins may style controls but must not suppress alarm dismiss/snooze affordances, privacy notices or VoiceOver labels. A future visual skin editor is not a V1 architectural prerequisite.

### 7.3 Dashboard widgets

Define a constrained `DashboardWidget` contract: identifier, capability/permission requirements, refresh policy, snapshot, freshness timestamp, loading/empty/error states and accessibility summary. Avoid independent timers or networking inside each SwiftUI view. A shared refresh scheduler coalesces requests, respects visibility and low-power mode, and marks stale data explicitly.

Calendar and weather providers remain optional adapters; absence of permissions or network should never prevent the core clock UI from starting.

### 7.4 Accessibility and layout

Support dynamic text where applicable, VoiceOver, reduced motion, high contrast, large touch targets, light/dark modes, portrait/landscape and iPad size classes. For the night clock, use a deliberately limited animation budget, periodic subtle movement or dimming as burn-in mitigations, and avoid claiming complete burn-in prevention.

## 8. Data architecture and migration

### 8.1 Data ownership

| Data | Proposed storage | Notes |
| --- | --- | --- |
| Profiles, skin selection, app settings | versioned local persistent store | atomic writes, migrations |
| Alarm definitions and OS mappings | durable local store | reconcile with OS on lifecycle changes |
| Media source catalog, favorites | durable local store | URLs may contain sensitive query tokens |
| Active playback / UI state | ephemeral in-memory state | persist only user-beneficial resume data |
| Imported skin assets | app-controlled files | validate, size-limit, atomic installation |
| Local media bookmarks | platform security-scoped references if needed | revalidate on access |
| Authentication secrets | Keychain | never export by default |
| Diagnostics | bounded local logs | redact tokens, paths and personal data |

**Storage engine decision is open:** `Codable` files may be sufficient for a small V1; SQLite/SwiftData/Core Data require explicit compatibility, migration and testing analysis. Do not bind domain entities to a particular persistence framework.

### 8.2 Versioning and transactions

- Every durable schema has an explicit version and migration path.
- Save with temporary file + atomic replace or database transaction, as appropriate.
- Validate before commit; preserve last-known-good configuration.
- Import uses **parse → validate → preview → commit**, with rollback on failure.
- Migration tests must include older snapshots, unknown fields, malformed files and interrupted writes.
- Export must be user initiated, documented and omit secrets by default.
- No cloud sync is required for V1. If added later, conflict resolution and encryption need a separate ADR.

### 8.3 Configuration precedence

Recommended precedence: platform constraints and safety rules > explicit user choices > active profile overrides > bundled defaults. Imported skins and routines must never override OS security constraints or alarm capability disclosures.

## 9. Security, privacy and permissions

### 9.1 Threat model

| Threat | Trust boundary | Mitigation |
| --- | --- | --- |
| Malicious playlist/feed or oversized response | network → parser | bounded parsing, timeouts, content validation |
| Hostile skin archive / path traversal | import → file system | normalized paths, safe extraction, size limits |
| Credential leakage in URLs/logs | app → diagnostics/export | redaction, Keychain, opt-in export |
| Unwanted LAN probing | imported URL → network | source trust policy, explicit user action |
| Unauthorized calendar access | OS permission → module | permission only when enabled and used |
| Unwanted sleep routine | settings → action | module disabled by default; separate session start |
| Misleading alarm state | OS adapter → UI | capability and delivery states kept distinct |
| Battery/thermal drain | app → device | adaptive refresh, pause hidden widgets, measurement |

Network access uses platform transport security by default. Exceptions for legacy HTTP radio streams, if needed, require a narrowly scoped documented policy and security review. No universal transport-security bypass.

### 9.2 Permissions

Request permissions **just in time** and explain the purpose before the OS prompt. Core clock rendering requires no calendar, microphone, camera, location or account permission. Weather may use a manually chosen location. Disabled modules should not initialize their data providers or run background jobs.

### 9.3 Observability

Structured logs with event categories (`alarm`, `media`, `profile`, `skin`, `migration`, `lifecycle`) and correlation IDs. No raw stream credentials, calendar event bodies, file contents or precise location in routine logs. User-facing diagnostics must distinguish planned intent from observed OS behavior. Remote telemetry is optional and off by default unless separately approved.

## 10. Device lifecycle, resource budgets and failure behavior

### 10.1 Lifecycle policies

Handle `active`, `inactive`, `background`, `suspended` and terminated states explicitly. The app must tolerate abrupt termination; there is no guaranteed shutdown callback. On foreground return, refresh time, re-evaluate permission/capability status, reconcile alarms, validate media state and mark stale widgets.

**Power and thermal strategy:** refresh seconds only when displayed; stop hidden animation; coalesce widget updates; use cached weather/calendar snapshots; avoid aggressive reconnect loops; observe system low-power and thermal signals where available. No busy polling to keep the app alive.

### 10.2 Failure containment

- Clock works if weather/calendar/radio fails.
- Bad skin falls back to bundled safe skin.
- Corrupt optional widget state does not block startup.
- Media failure does not erase saved source.
- Alarm scheduling failure remains visible and retryable; never report false success.
- Storage migration failure must not silently reset user data.

### 10.3 Quality targets and evidence

Numerical performance thresholds are **not yet approved**. P0 must establish baselines on a representative oldest supported iPhone and iPad: cold start, memory, idle CPU, energy impact, heat, 8–24-hour display endurance, network recovery and UI responsiveness. Targets should then be documented as measurable budgets with device/OS/method and pass/fail criteria.

## 11. Compatibility and portability strategy

**V1:** Swift/SwiftUI native Apple application with availability checks around APIs newer than iOS 16. Avoid accidentally introducing a higher deployment target through packages, macros or UI APIs. Verify with the actual supported Xcode/toolchain matrix.

**Future:** platform-neutral domain rules and serializable contracts may later move to a C++20 library or be reimplemented with conformance tests. Do not introduce C++ solely for hypothetical reuse. Apple playback, alarm delivery, permissions and UI remain platform-specific by definition.

**Portability seam:** `ClockPort`, `AlarmDeliveryPort`, `MediaPlaybackPort`, `PersistencePort`, `PermissionPort`, `WidgetDataPort`. Prefer a small, stable interface surface and behavior tests over broad generic abstractions.

## 12. Repository structure (proposal, not current tree)

```text
SASD-Multimedia-Clock/
├── README.md
├── AGENTS.md                         # planned
├── docs/
│   ├── ARCHITECTURE.md               # this document
│   ├── ROADMAP.md                    # planned
│   ├── adr/
│   ├── requirements/
│   ├── research/
│   ├── testing/
│   └── images/
├── App/                               # proposed native app
│   ├── AppShell/
│   ├── Features/
│   │   ├── Clock/
│   │   ├── Alarms/
│   │   ├── Media/
│   │   ├── Profiles/
│   │   ├── Dashboard/
│   │   └── Settings/
│   ├── Domain/
│   ├── Application/
│   ├── Infrastructure/
│   └── Resources/
├── Tests/
│   ├── DomainTests/
│   ├── AdapterContractTests/
│   ├── IntegrationTests/
│   └── UITests/
└── .github/workflows/                 # planned
```

Prefer feature-first grouping within the app while keeping the dependency direction enforceable. Do not create empty packages for speculative modules.

## 13. Testing strategy and release gates

### 13.1 Test pyramid

- **Pure unit tests:** recurrence, time zones/DST, profile inheritance, skin validation, URL classification, state transitions, idempotency.
- **Contract tests:** fake clock, fake alarm delivery, fake playback, storage and permission adapters.
- **Integration tests:** real OS notification scheduling, AVFoundation stream playback, persistence migrations, import rollback.
- **UI tests:** profile switching, alarm permission disclosures, accessible controls, compact and regular layouts.
- **Device tests:** real iPhone/iPad, iOS 16 minimum where feasible, current iOS, locked/background/force-quit/reboot, long-running power and heat.

Do not count mocks as proof of OS behavior.

### 13.2 Gate P0 — feasibility

**Exit criteria (all evidenced):**
1. Reproducible build and launch on the agreed minimum OS and representative devices.
2. Direct audio stream plays and failure paths are observable.
3. Alarm delivery matrix completed with limitations documented, including locked/terminated cases.
4. Video feasibility separated into foreground playback vs background/locked autoplay.
5. Representative bedside and desk screens run acceptably on older devices.
6. Documented decision on the actual V1 alarm promise.

### 13.3 Gate ARCH — architecture approval

- Module boundaries and ownership reviewed.
- Alarm semantics and failure reporting approved.
- Data model/migration strategy chosen.
- Security/privacy threat model reviewed.
- iOS deployment target and toolchain validated.
- ADRs for open decisions recorded and accepted.
- Traceability to requirements and test plan reviewed.

### 13.4 Gate V1 — release

- Critical requirements have tests and reproducible evidence.
- No misleading alarm reliability claims.
- Offline, permissions and data migration tested.
- Accessibility and older-device endurance accepted.
- Legal/stream rights and privacy disclosures reviewed.
- App Store submission prerequisites verified independently.

## 14. Architectural decision record (ADR) backlog

Create individual ADRs under `docs/adr/` as decisions mature. Each ADR must state context, alternatives, decision, consequences, evidence, status and supersession rules.

| ADR | Decision | Proposed direction | Gate |
| --- | --- | --- | --- |
| ADR-001 | Native Apple implementation | Swift + SwiftUI | P0 |
| ADR-002 | Minimum deployment target | iOS/iPadOS 16 if validated | P0 |
| ADR-003 | Alarm API and delivery promise | capability-gated platform adapter | P0 |
| ADR-004 | Local persistence engine | compare Codable files vs database | ARCH |
| ADR-005 | Concurrency and dependency injection | structured concurrency + composition root | ARCH |
| ADR-006 | Skin format and import trust | versioned declarative schema | ARCH |
| ADR-007 | Media resolver and URL security | typed sources, bounded resolver | ARCH |
| ADR-008 | Playback/background audio policy | explicit audio-session coordinator | P0 |
| ADR-009 | Profile/global settings ownership | explicit ownership matrix | ARCH |
| ADR-010 | Dashboard refresh and data permissions | visibility-aware shared refresh | ARCH |
| ADR-011 | Telemetry and diagnostics | local, redacted by default | ARCH |
| ADR-012 | Shared C++ core | defer pending concrete second platform | Later |
| ADR-013 | Licensing and third-party assets | decision required before release | V1 |
| ADR-014 | Sleep assistance opt-in contract | disabled by default, explicit start | ARCH |

## 15. Requirements traceability

This document complements rather than replaces the requirements IDs in the [Lastenheft](requirements/Lastenheft-v0.1.md) and [Pflichtenheft](requirements/Pflichtenheft-v0.1.md).

| Requirement family | Architecture sections | Primary verification |
| --- | --- | --- |
| CLK | 4, 5, 7, 10 | time-zone, rendering and lifecycle tests |
| ALM | 4, 5, 9, 13 | real-device alarm matrix |
| AUD / MUS / POD | 4, 6, 8, 9 | source, codec, offline and failure tests |
| VID | 5, 6, 11, 13 | foreground vs locked-state feasibility |
| SLP | 4, 7, 9 | opt-in and cancellation tests |
| TMR / WLD / PRD | 4, 5, 7 | time semantics and UI tests |
| DSH | 4, 7, 10 | stale data, permissions, refresh tests |
| SKN | 4, 7, 8, 9 | malicious import, rollback, accessibility |
| AUT | 4, 5, 10 | idempotency, cancellation, capability tests |
| Platform / older devices | 1, 10, 11, 13 | iOS 16 compatibility, endurance |

**Traceability rule:** each approved V1 requirement should ultimately link to a module owner, a test ID, a test artifact and a release-gate decision. Unverified features remain labeled planned.

## 16. Architecture review checklist

- [ ] Product scope and V1 promises approved by the project owner.
- [ ] Alarm and routine semantics tested and signed off on physical devices.
- [ ] iOS 16 compatibility validated with an actual supported toolchain.
- [ ] Global versus profile-scoped settings defined.
- [ ] Storage engine and migration strategy chosen.
- [ ] Skin trust boundary and import limits implemented/tested.
- [ ] Media URL security and privacy model accepted.
- [ ] Lifecycle and background behavior documented without unsupported guarantees.
- [ ] Minimum-device energy and thermal budgets established.
- [ ] Accessibility acceptance criteria written.
- [ ] ADRs created for accepted architectural decisions.
- [ ] Roadmap and AGENTS.md derived **after** architectural approval.

## 17. Known uncertainties and explicit non-guarantees

1. **Alarm behavior:** the exact capabilities of each supported OS release and entitlement must be proven with public API documentation and physical-device tests. A notification scheduling success does not guarantee an audible wake-up.
2. **Automatic television playback:** a locked or terminated app cannot be assumed to launch arbitrary video; treat this as conditional research, not a committed feature.
3. **Stream availability:** a working URL today is not a service-level guarantee.
4. **Old devices:** iOS 16 compatibility is a target, not a verified build matrix.
5. **App Store distribution:** requires current toolchain, signing, privacy and review compliance.
6. **Cross-platform:** future portability is an option; no second implementation exists yet.
7. **Performance:** thresholds remain hypotheses until measured on target hardware.

---

**Next document:** `docs/ROADMAP.md`, after review of this architecture and resolution of the P0-critical ADRs. The future `AGENTS.md` should enforce these boundaries, not invent new architecture independently.
