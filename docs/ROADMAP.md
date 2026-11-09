# SASD Multimedia Clock — Development Roadmap

| Field | Value |
| --- | --- |
| Document ID | SASD-MC-RM-001 |
| Version | 0.1 |
| Date | 2026-11-09 |
| Status | **Proposed — planning baseline, not a delivery commitment** |
| Source of truth | [Architecture](ARCHITECTURE.md), [Lastenheft](requirements/Lastenheft-v0.1.md), [Pflichtenheft](requirements/Pflichtenheft-v0.1.md) |
| Engineering rules | [AGENTS.md](../AGENTS.md) |

> **Current reality:** requirements and architecture exist; this roadmap does **not** assert that an Xcode application, automated tests, validated alarm delivery, or production-ready features already exist. Each milestone is **planned** until independently verified and documented.

## 1. Product direction and planning rules

SASD Multimedia Clock will repurpose older working iPhones and iPads as configurable bedside clocks, clock radios, desk dashboards and multimedia stations. Its first implementation is a universal native Swift/SwiftUI app. A later portable C++20 domain core is an option, not a V1 prerequisite.

**Non-negotiable principles:** local-first; no required account; modular opt-in features; safe declarative skins; explicit capability/permission disclosures; no unproven background alarm or video claims; measurable performance on the oldest supported devices.

**Planning convention:** P0 is a feasibility gate, M1–M6 are proposed implementation milestones. Their scope and order may be adjusted after P0 evidence. The term “complete” requires the exit criteria, not merely merged code.

### 1.1 Priority levels

- **P0-blocker:** unknown feasibility or platform behavior that could invalidate a product promise.
- **Must:** needed for the agreed V1 release slice after architecture approval.
- **Should:** useful for V1 if risk and schedule allow; otherwise V1.x.
- **Could:** deferred unless it has negligible cost.
- **Research:** explicitly not a delivery commitment.

### 1.2 Milestone overview

| Phase | Outcome | Key dependency | State |
| --- | --- | --- | --- |
| **P0** | iOS 16/device, alarm, audio/video, UI feasibility evidence | available Apple build/test environment | Planned |
| **ARCH** | architecture and P0-critical ADR decisions accepted | P0 evidence | Planned |
| **M1** | app shell, time display, profiles, settings and persistence | ARCH | Planned |
| **M2** | playable audio sources, favorites and recovery | M1 | Planned |
| **M3** | truthful alarm workflows, timers and notification UX | P0, M1 | Planned |
| **M4** | validated skins, layout presets and accessibility | M1 | Planned |
| **M5** | foreground video, conditional routines, optional sleep assistance | M2–M4 | Planned |
| **M6** | compatibility, endurance, security and release verification | M1–M5 | Planned |
| **V1.0** | approved, supportable release | all release gates | Not approved |
| **V1.x / V2+** | podcasts, advanced widgets, integrations, editor, ports | post-V1 decisions | Research |

**Critical path:** Apple toolchain/device access → P0 alarm/OS feasibility → agreed V1 claims → domain/data architecture → implementation → real-device acceptance. Cosmetic work must not mask unresolved alarm feasibility.

## 2. P0 — Technical feasibility (must precede V1 promises)

**Goal:** replace assumptions with reproducible evidence. P0 is not a consumer release.

### P0.1 — Apple toolchain and deployment

**Tasks**
1. Inventory available macOS/Xcode versions and physical test devices; record model, processor, RAM, iOS/iPadOS version and update status.
2. Determine a current build toolchain that supports the desired minimum deployment target and future distribution.
3. Create a disposable universal SwiftUI proof of concept, with iPhone/iPad targets and explicit deployment target.
4. Verify **build + launch on a real minimum-version device** where feasible, not only simulator.
5. Record unsupported APIs and feature gates for newer OS versions.

**Evidence:** toolchain versions, build logs, device screenshots, reproducible setup instructions, compatibility table.

**Exit:** explicit go/no-go decision on iOS/iPadOS 16.0. Do not silently raise the minimum version.

### P0.2 — Alarm delivery truth table (**highest product risk**)

**Tasks**
1. Compare publicly supported platform notification/alarm facilities by OS version.
2. Build a small test harness that distinguishes alarm intent, OS scheduling, delivery observation and user acknowledgement.
3. Test foreground, background, locked, suspended, force-quit, restart, offline, denied permissions, Focus/Do Not Disturb, silent mode, low-power mode, Bluetooth route change, clock/time-zone change and DST.
4. Record what can and cannot be guaranteed, including any API-specific eligibility or OS-version restrictions.
5. Design user-facing fallback and degraded-state messages; decide whether the product may use “alarm clock” wording on each supported OS.

**Evidence:** device/OS/API matrix, test procedure, observed results, log timestamps, screenshots and known limitations.

**Exit:** **ADR-003 accepted** and a user-visible alarm promise backed by physical-device results. Scheduling an OS notification is not proof that the user was audibly awakened.

### P0.3 — Media capability tests

**Tasks**
1. Validate direct HTTPS MP3/AAC and, where applicable, HLS with AVFoundation.
2. Verify a sample SWR3 URL as a **test fixture only**, not a permanent service guarantee.
3. Classify unsupported codecs, playlist redirects, malformed URLs, timeouts, TLS failures and reconnect.
4. Verify local audio import and authorization lifecycle.
5. Test foreground video; separately attempt locked/background/terminated scenarios to document limitations.
6. Test audio interruptions, route changes and whether background audio is permissible under declared capabilities.

**Evidence:** source-by-source playback matrix, network conditions, device/OS, logs and failure screenshots.

**Exit:** supported media-source list and truthful background playback policy; **ADR-008 accepted**.

### P0.4 — UI, accessibility and older-device endurance

**Tasks**
1. Implement a minimal bedside clock and desk dashboard, with large text and an accessible control path.
2. Measure foreground idle CPU, memory, energy impact, startup and thermal behavior on the oldest supported iPhone and iPad.
3. Exercise portrait/landscape, iPad multitasking, VoiceOver, reduced motion and low-light operation.
4. Evaluate screen-dimming and burn-in mitigation without promising prevention.
5. Document the implications of long-term charging and obsolete security updates.

**Evidence:** repeatable measurement protocol and baseline results. Numeric targets are established **after** measurement.

**Exit:** feasible UI/performance baseline or documented reduction of scope.

### P0.5 — P0 decision package

Produce `docs/testing/P0-RESULTS.md` (when tests have actually run), including:
- environment and device matrix;
- every test's expected vs observed behavior;
- linked logs/evidence;
- unresolved risks and mitigations;
- accepted/rejected claims;
- decision on minimum OS and V1 alarm feature;
- explicit owner/date for open issues.

**Gate P0:** all four investigations completed and reviewed; **no unverified item marked passed**.

## 3. ARCH — Architecture review and ADRs

**Goal:** convert [ARCHITECTURE.md](ARCHITECTURE.md) from proposal into an approved implementation baseline.

| ADR | Topic | Required before |
| --- | --- | --- |
| ADR-001 | Native Swift/SwiftUI | M1 |
| ADR-002 | iOS/iPadOS deployment target | M1 |
| ADR-003 | alarm API and user promise | M3 |
| ADR-004 | persistence and migrations | M1 persistence |
| ADR-005 | concurrency and dependency injection | M1 |
| ADR-006 | skin schema/trust | M4 |
| ADR-007 | media resolver and URL security | M2 |
| ADR-008 | playback and background audio | M2 |
| ADR-009 | global vs profile-scoped settings | M1 |
| ADR-010 | widget refresh and permissions | M5 widgets |
| ADR-011 | privacy and diagnostics | M1 |
| ADR-012 | shared C++ core | later, **defer** |
| ADR-013 | license and third-party assets | V1 |
| ADR-014 | sleep-assistance opt-in contract | M5 |

Each accepted ADR must contain context, alternatives, decision, consequences, evidence, status and supersession notes. Proposed decisions are not binding until reviewed.

**Exit:** approved module ownership, capability contracts, data model, security boundary and testing approach; all critical ADRs accepted.

## 4. M1 — Foundation, clock and profiles

**Outcome:** a usable local clock application, not yet a guaranteed alarm clock.

**Implementation slices**
- **M1.1 Repository/bootstrap:** Xcode project, app ID placeholder strategy, Swift formatting/lint configuration, unit/UI test targets, reproducible build documentation, CI baseline.
- **M1.2 Domain and composition root:** pure time/profile models; injected clock, storage and permission ports; no hidden global mutable state.
- **M1.3 Clock UI:** digital/analog variants, 12/24-hour format, optional seconds/date, time-zone handling, adaptive rendering.
- **M1.4 Profiles:** bedside, desk, multimedia, custom; explicit global-vs-profile ownership; switching without loss.
- **M1.5 Persistence:** versioned local settings, atomic saves, migration fixtures, corrupted-data fallback and import/export policy.
- **M1.6 Settings/accessibility:** clear navigation, localized strings (English/German design), VoiceOver, contrast, text sizing and night display options.
- **M1.7 Lifecycle:** scene changes, clock refresh throttling, foreground recovery, no unnecessary background work.

**Acceptance**
- Clock works without network or optional permissions.
- Profile changes do not erase alarms/favorites.
- DST/time-zone tests and migration tests pass.
- Minimum-device smoke tests pass.
- UI has no hidden network dependency.

## 5. M2 — Audio, radio and media sources

**Implementation slices**
- **M2.1 Typed source catalog:** direct audio, local file, playlist reference; schema validation; favorites.
- **M2.2 Resolver:** URL classification, redirect limits, safe parsing, typed errors, HTTPS policy.
- **M2.3 Playback coordinator:** idle/resolving/preparing/playing/paused/buffering/failed/stopping transitions, one primary media session.
- **M2.4 Audio-session policy:** interruptions, Bluetooth/headphones, remote route changes, eligible background audio.
- **M2.5 Recovery:** bounded reconnect, cancellation, offline status, unsupported formats, explicit diagnostics.
- **M2.6 Persistence:** stable source IDs and no credential leakage in export/logs.

**Acceptance**
- Supported direct streams play on physical devices under stated conditions.
- Bad URLs/codec failures are understandable and do not crash the app.
- Offline local audio works where authorized.
- Reconnect and route-change tests pass.
- No unsupported claim of locked-screen scheduled startup.

**Defer:** DRM-protected streaming providers, arbitrary webpage scraping, recording protected content.

## 6. M3 — Alarm and timer workflows

**Dependency:** P0.2 results and ADR-003. Do not implement an unreliable fake system alarm.

**Implementation slices**
- **M3.1 Alarm model:** named alarms, weekday recurrence, time-zone policy, DST semantics.
- **M3.2 Platform adapter:** capability detection, permission UX, OS schedule/cancel/reconcile.
- **M3.3 Honest state UI:** configured, permission-granted, scheduled, degraded, observed and acknowledged are distinct.
- **M3.4 Snooze/dismiss:** only where OS APIs and app execution allow; document constraints.
- **M3.5 Fallback:** locally available alert where supported; explicit failure states.
- **M3.6 Timer:** countdown and stopwatch with monotonic time semantics; documented restoration behavior.
- **M3.7 Device matrix:** repeat P0 critical tests against integrated app.

**Acceptance**
- Each alarm has a persisted intent and a visible reconciliation status.
- No success claim based only on a scheduled notification.
- Physical-device locked/background/reboot evidence supports all user-facing promises.
- Recurrence and DST behavior is deterministic and tested.
- Denied permission and unsupported OS capability have clear UI.

## 7. M4 — Skin engine and layout system

**Implementation slices**
- **M4.1 Design tokens:** bundled classic-radio, minimalist and dark presets.
- **M4.2 Declarative manifest:** schema version, IDs, tokens, allowed layout slots, package-relative assets.
- **M4.3 Validation/import:** size limits, path traversal prevention, rollback, unknown-version rejection.
- **M4.4 Renderer:** adaptive iPhone/iPad layouts and accessible semantics independent of skin.
- **M4.5 Profile binding:** skin choice per profile, safe fallback after corruption.
- **M4.6 Testing:** malicious/oversized asset fixtures, contrast and VoiceOver.

**Acceptance**
- Switching skins never deletes user configuration.
- Invalid skin cannot crash startup or execute code.
- Alarm controls and accessibility remain usable under every bundled skin.
- Resource consumption is measured on older devices.

**Defer:** visual skin editor, arbitrary scripting, legacy Winamp skin compatibility.

## 8. M5 — Multimedia, conditional routines and optional modules

**Scope must be re-evaluated after P0.** Some items may move to V1.x.

- **M5.1 Foreground video:** supported direct URLs/local files, playback controls, error handling.
- **M5.2 Routine engine:** typed steps, eligibility checks, idempotency, cancellation, timeouts and outcome reporting.
- **M5.3 Morning routine:** optional alarm-related follow-up media **only when execution is allowed**; no promised locked-screen TV autoplay.
- **M5.4 Sleep assistance:** module **off by default**; user must explicitly enable it **and start a session**; timer/fade/cancel semantics.
- **M5.5 Desk widgets (conditional):** world clock, weather/calendar adapters with explicit permissions and stale-state UI.
- **M5.6 Media extensions (conditional):** podcasts and feed parsing if time and risk permit.

**Acceptance**
- Routine actions never bypass OS restrictions.
- Duplicate triggers do not duplicate playback.
- Sleep assistance cannot start implicitly.
- Optional module failures do not break the clock.
- Calendar/weather modules do not request permissions while disabled.

## 9. M6 — Hardening, compatibility and release

**Workstreams**
1. **Reliability:** crash recovery, corrupt settings, stream outage, permission changes, migration and alarm reconciliation.
2. **Performance:** measure startup, CPU, memory, energy, thermal and 8–24-hour display use on oldest supported devices; approve budgets.
3. **Accessibility:** VoiceOver, high contrast, large touch targets, reduced motion, dynamic text, portrait/landscape and iPad multitasking.
4. **Security:** URL/feed parsing, imported skins, token redaction, Keychain, privacy review and transport-security exceptions.
5. **Distribution:** signing, entitlements, current Xcode, App Store privacy information, age rating, third-party licenses and screenshots.
6. **Documentation:** README accuracy, supported device matrix, release notes, known limitations, troubleshooting, source/skin format documentation.
7. **Release candidate:** reproducible tagged build, test report, artifact hashes where applicable, explicit approval record.

**V1 release gate:** no critical unverified claims; requirements traceability and device evidence complete; license chosen; all blocking defects resolved or explicitly excluded from release scope.

## 10. Cross-cutting verification and traceability

### 10.1 Requirement families

| Family | Owner module | First milestone | Evidence |
| --- | --- | --- | --- |
| CLK | ClockDomain / UI | M1 | DST, rendering, lifecycle |
| ALM | AlarmDomain / AlarmCoordinator | P0, M3 | physical-device delivery matrix |
| AUD/MUS | MediaResolver / PlaybackCoordinator | P0, M2 | stream and local-file matrix |
| POD | MediaCatalog / feed adapter | M5 or V1.x | feed parsing and offline |
| VID | PlaybackCoordinator | P0, M5 | foreground/locked-state separation |
| SLP | SleepAssistance | M5 | opt-in and cancellation |
| TMR/WLD/PRD | Timer / Clock / Dashboard | M3–M5 | monotonic time and UI |
| DSH | Dashboard | M5 | stale data, permissions, refresh |
| SKN | SkinSystem | M4 | schema, rollback, accessibility |
| AUT | RoutineEngine | M5 | idempotency, capability checks |

Map individual requirement IDs from the specifications to test IDs as implementation starts; this table is a starting point, not a substitute for the full traceability matrix.

### 10.2 Standard feature definition of done

A feature is complete only when:
- requirements and acceptance criteria are linked;
- code respects architectural boundaries and platform availability;
- positive, negative, lifecycle and permission cases are tested;
- accessibility and localization implications are handled;
- persistence migrations are covered if data shape changes;
- logs redact secrets and errors are user-readable;
- CI passes and real-device evidence exists where OS behavior matters;
- documentation and known limitations are updated;
- reviewer or owner accepts the result.

### 10.3 Evidence record template

```markdown
## Evidence: <ID>
- Requirement / milestone:
- Device and OS:
- Xcode / build / commit:
- Test procedure:
- Expected:
- Observed:
- Result: PASS | FAIL | BLOCKED | DOCUMENTED_ONLY
- Logs/screenshots/artifacts:
- Known limitations:
- Reviewer and date:
```

**DOCUMENTED_ONLY is never equivalent to PASS.** Unknown or untested cases remain open.

## 11. Risk register

| Risk | Impact | Response | Owner / gate |
| --- | --- | --- | --- |
| iOS cannot satisfy proposed alarm expectations | Critical | P0 physical-device testing; narrow promise | P0 / ADR-003 |
| Old Mac cannot run required Xcode | High | toolchain inventory, newer build host if needed | P0 |
| iOS 16 limits required APIs | High | capability-gated implementations; revise scope | P0 / ADR-002 |
| Legacy devices overheat during long display use | High | measured refresh/brightness/power policies | P0, M6 |
| Streaming URLs/rights change | Medium | typed errors, configurable sources, legal review | M2 |
| User skin package is malicious | High | data-only schema, bounded import, rollback | M4 |
| Scope grows beyond V1 | High | strict must/should/could and gate approval | all |
| Data loss during migrations | High | atomic writes, fixtures, rollback | M1 |
| Optional integrations leak personal data | High | just-in-time permissions, local-first | M5/M6 |

## 12. Initial issue backlog (ready to convert to GitHub issues)

| ID | Proposed issue title | Priority | Dependency |
| --- | --- | --- | --- |
| P0-01 | Inventory Apple development host and test devices | Blocker | none |
| P0-02 | Prove iOS 16 universal app deployment | Blocker | P0-01 |
| P0-03 | Build alarm delivery feasibility harness | Blocker | P0-02 |
| P0-04 | Execute alarm device-state matrix | Blocker | P0-03 |
| P0-05 | Validate direct audio and local file playback | Blocker | P0-02 |
| P0-06 | Validate foreground vs locked video behavior | Blocker | P0-02 |
| P0-07 | Measure oldest-device clock/dashboard baseline | Blocker | P0-02 |
| ARCH-01 | Decide ADR-001/002/003/008 from P0 evidence | Blocker | P0 |
| ARCH-02 | Decide storage, DI, profile and skin contracts | Must | P0 |
| M1-01 | Bootstrap app, tests and CI | Must | ARCH |
| M1-02 | Implement clock and profile state | Must | M1-01 |
| M1-03 | Implement versioned settings and migrations | Must | M1-01 |
| M2-01 | Implement typed media sources and resolver | Must | M1 |
| M2-02 | Implement playback coordinator and errors | Must | M2-01 |
| M3-01 | Implement OS alarm adapter and reconciliation | Must | ADR-003 |
| M4-01 | Implement safe declarative skin renderer | Must | ADR-006 |
| M5-01 | Implement conditional routines and opt-in sleep | Should | M2/M3 |
| M6-01 | Complete older-device endurance and security tests | Must | M1–M5 |

No issues, branches or code are created by this roadmap alone.

## 13. Release scope decisions still open

- Does V1 ship with a full alarm experience on iOS 16, a limited notification-based reminder, or a higher minimum OS? **Decide only after P0.**
- Which dashboard integrations are V1 must-haves?
- Is foreground video V1 or V1.x?
- Which source formats are officially supported?
- Which persistence engine and skin manifest version become stable?
- Which license and asset policies apply?

## 14. Next actions

1. Confirm access to an Apple development environment and representative older iPhone/iPad hardware.
2. Create the P0 test harness and evidence log **before** implementing full alarm features.
3. Review and accept P0-critical ADRs; update [ARCHITECTURE.md](ARCHITECTURE.md) accordingly.
4. Convert the issue backlog to GitHub issues/milestones after scope approval.
5. Start M1 with a small, buildable native app and automated tests.

---

*This roadmap is intentionally evidence-gated. No milestone is considered complete merely because it appears in this document.*
