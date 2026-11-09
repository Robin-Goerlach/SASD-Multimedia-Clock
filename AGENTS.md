# AGENTS.md — SASD Multimedia Clock

**Applies to:** the entire repository, including documentation, tests, assets and future Swift/Xcode source. A more specific nested `AGENTS.md` may refine rules for its subtree but must not contradict these repository-wide safety and architecture requirements.

**Project stage:** requirements and architecture planning. Do not assume a buildable app, existing CI, passing tests, approved license, or completed milestones unless verified in the current repository.

## 1. Mission and source-of-truth hierarchy

Build a reliable, accessible, local-first universal iPhone/iPad clock, alarm-radio and multimedia dashboard that extends the useful life of older Apple devices.

Before changing implementation or scope, read:
1. [Architecture](docs/ARCHITECTURE.md) — boundaries, risks, technical proposals.
2. [Roadmap](docs/ROADMAP.md) — milestone order, acceptance and evidence gates.
3. [Lastenheft](docs/requirements/Lastenheft-v0.1.md) — product needs.
4. [Pflichtenheft](docs/requirements/Pflichtenheft-v0.1.md) — proposed technical behavior.
5. [README](README.md) — public claims; must remain accurate.

**Conflict resolution:** platform security and real observed behavior > approved ADRs > approved acceptance criteria > architecture baseline > proposed technical specification > roadmap implementation suggestions > README prose. Requirements disagreements must be surfaced and resolved, not silently ignored. Draft documents are **not** automatically approved decisions.

## 2. Hard product constraints

- One universal native iPhone/iPad app. Initial direction: Swift + SwiftUI, with AVFoundation and OS APIs behind adapters.
- Target older functioning devices; **iOS/iPadOS 16.0 is proposed, not validated**. Do not raise deployment target without an explicit documented decision.
- Profiles: **Bedside, Desk, Multimedia, Custom**. Switching profiles must not delete global alarms or favorites.
- Optional sleep assistance is **disabled by default** and additionally requires explicit user session start.
- Local-first; no mandatory cloud account, telemetry, tracking or unrelated permissions.
- Declarative, versioned skins only. No arbitrary executable code or remote scripts in skins.
- Do not introduce a C++20 core, dynamic plugin runtime, third-party SDK or cloud backend merely for hypothetical future portability.
- Treat all screenshots and visual mockups as **concept art** until backed by a running build.

## 3. Alarm and background-execution rules (critical)

**Never claim that a third-party iOS app can reliably wake/unlock a suspended or terminated device and start arbitrary streaming audio/video at a chosen time.** Background audio eligibility does not imply background autoplay or native alarm guarantees.

Maintain separate concepts:
- persisted **alarm intent**;
- permission and capability status;
- OS scheduling result;
- observed delivery (if observable);
- user acknowledgement;
- optional **routine** actions that run only when eligible.

A successful scheduling API call is **not** evidence that the alarm sounded. Do not relabel a notification as a guaranteed system alarm. Provide explicit degraded/unsupported states and user-facing explanations. Never bypass OS permissions, private APIs, DRM or provider restrictions.

**Required before alarm claims:** physical-device tests for foreground/background/locked/force-quit/reboot/offline/permission denial/Focus/silent mode/time-zone and DST transitions. Simulator-only evidence is insufficient. Follow P0 and ADR-003.

## 4. Architecture and code organization

Follow a modular monolith with ports/adapters:

```text
SwiftUI presentation → application use cases → pure domain
                                  ↓ protocols
                         platform/infrastructure adapters
```

- Domain must not import SwiftUI, UIKit, AVFoundation or concrete storage frameworks.
- UI renders state and dispatches typed commands; no direct network, notification or persistence calls from views.
- Application services coordinate workflows and capability checks; adapters own OS/network/storage mechanics, not product policy.
- Compose dependencies in an explicit composition root. Avoid global mutable singletons and cross-feature import cycles.
- Prefer feature-focused folders and small protocols with test doubles. Do not create empty abstraction layers without a real use case.
- Use structured Swift concurrency and cancellation. Isolate shared mutable state (actors/main actor as appropriate). Do not block the main actor with decoding, disk or network work.
- Use availability checks for APIs introduced after the minimum deployment target. Do not hide compiler or runtime incompatibilities behind assumptions.

**Suggested module responsibilities:** ClockDomain, ProfileService, AlarmDomain/AlarmCoordinator, RoutineEngine, MediaCatalog/MediaResolver/PlaybackCoordinator, SkinSystem, Dashboard, SleepAssistance, SettingsStore, Diagnostics and PlatformAdapters. See architecture document for ownership and state machines.

## 5. Swift engineering conventions

- Use clear Swift names and types; prefer namespaces/modules over redundant long prefixes. No forced `Sasd` prefix on every symbol.
- Prefer readable, well-commented code over clever abstractions. Comments explain invariants, constraints and **why**, not restate obvious syntax.
- Make invalid states difficult to represent: typed IDs, enums for state/error categories, validated constructors and explicit optionality.
- Use `async/await` for asynchronous operations; define cancellation, timeout and retry semantics. No unbounded retries or busy loops.
- Time: separate wall-clock recurrence from elapsed/monotonic durations. Use Calendar/TimeZone; test DST gaps and repeated hours.
- Keep persistence models versioned and separate from UI models. Handle schema migrations and failed writes.
- Strings for UI must be localizable; English and German are intended. Never hard-code untranslatable strings in views.
- Avoid speculative dependencies; explain any added package, license, minimum OS impact, binary size and maintenance burden.
- Do not commit credentials, provisioning profiles, personal data, generated build products or machine-specific paths.

## 6. Media and security requirements

- Treat external URLs, playlist files, podcast feeds, imported configuration and skins as untrusted data.
- Use typed media-source kinds; a webpage URL is not necessarily a playable media URL.
- Enforce timeouts, redirect and response-size limits, safe parsing, path normalization and bounded retries.
- Prefer HTTPS and platform transport security. Any HTTP exception requires a scoped, reviewed justification; never globally disable security.
- Never attempt DRM circumvention or unsupported streaming-provider integration.
- Store credentials in Keychain where needed; redact tokens, query secrets and sensitive paths from logs and exports.
- Local media access must respect sandbox/security-scoped permissions.
- Import skins via validate → preview → atomic commit; reject path traversal, remote scripts and oversized assets; retain last-known-good skin.
- Request calendar/location/notification permissions only when the relevant feature is enabled and used. A clock should launch without optional permissions.
- Disable network work and refresh timers for inactive optional modules.

## 7. Performance, lifecycle and accessibility

- The oldest supported physical iPhone and iPad are first-class test targets.
- Do not keep the app alive by busy polling. Throttle clock rendering by visibility and granularity; coalesce dashboard refresh.
- Handle scene transitions, interruption, route changes, offline state, clock changes and abrupt termination. Do not rely on a shutdown callback.
- Avoid high CPU, thermal stress, excessive brightness and burn-in risks during long-running bedside operation. Measure, do not invent performance figures.
- Maintain VoiceOver labels, high contrast, reduced-motion behavior, large targets and adaptable layouts. Skins must not remove essential controls or accessible semantics.
- A failure of weather, radio or optional widgets must not prevent the basic clock from working.

## 8. Tests and evidence are part of the feature

Each change should add or update relevant tests:

| Layer | Examples |
| --- | --- |
| Pure unit | DST/recurrence, profile precedence, validation, state transitions |
| Contract | fake clock, alarm adapter, media player, persistence |
| Integration | AVFoundation, notifications, local file access, migrations |
| UI | profile switching, error messages, VoiceOver, device layouts |
| Physical device | locked/background/reboot alarm behavior, power, thermal, audio routes |

**Evidence statuses:** `PASS`, `FAIL`, `BLOCKED`, `DOCUMENTED_ONLY`. Never equate documented-only with passed. Every OS-specific claim requires reproducible device/OS/build evidence. Tests must not depend on a live third-party stream unless explicitly designated as a non-deterministic integration test; use fixtures/fakes for deterministic CI.

For every completed feature, provide: requirement ID, test IDs, actual commands/results, supported device/OS, known limitations and updated docs. If tests cannot run because the host lacks Xcode or hardware, state **not run**, never “passed”.

## 9. Workflow for coding agents

1. **Inspect first:** check the actual repository tree, branch, working tree, existing changes and applicable AGENTS/ADRs.
2. **Define scope:** cite the requirement and roadmap slice; distinguish approved behavior from open research.
3. **Plan the smallest coherent change:** include tests, error paths, permissions, migration and documentation.
4. **Implement incrementally:** preserve boundaries and existing user changes. Avoid unrelated refactors.
5. **Validate:** run available format/lint/build/tests; document exact results and unavailable checks.
6. **Review risk:** confirm no unsupported alarm claims, API availability regressions, secret exposure or unwanted background behavior.
7. **Document:** update README, requirements traceability, architecture/ADR or roadmap if the behavior/decision changed.
8. **Report:** summarize files, behavior, evidence, risks and next action. Never claim an unperformed merge or deployment.

**Branch policy:** work on the user's requested branch. Direct-to-`main` changes are permitted when explicitly requested and the operation is safe; do not force-push, rewrite history or discard other contributors' changes. Do not create PRs or merge without a relevant instruction. For high-risk architectural changes, request/record an ADR before implementation.

**Scope policy:** do not treat a broad task as permission to implement unrelated roadmap phases. Ask for clarification when a decision would change OS support, public alarm guarantees, licensing or security boundaries.

## 10. Documentation conventions

- **README and engineering documents:** English by default; existing German Lastenheft/Pflichtenheft are authoritative inputs and should not be silently translated or overwritten.
- Keep Markdown headings, relative links and terminology consistent.
- New ADRs: `docs/adr/ADR-NNN-short-title.md` with `Status: Proposed | Accepted | Superseded | Rejected`, date, context, alternatives, decision, consequences, tests and references.
- New evidence: `docs/testing/` with device model, OS, Xcode/build/commit, procedure, expected/observed, result, artifacts and limitations.
- Never invent screenshots, performance numbers, CI badges, feature completeness or test results.
- Do not mark roadmap checkboxes complete without linked evidence.
- Any modification to architectural contracts must update `docs/ARCHITECTURE.md` and the relevant ADR.

## 11. Definition of done and release gates

A task is **done** only when implementation and tests are verified, docs match reality, security/accessibility implications are addressed, and any platform limitation is disclosed. A feature with no hardware evidence may be **implemented but unverified**, not production-ready.

**P0:** minimum-OS/toolchain, alarm truth table, media capability and older-device endurance evidence.

**ARCH:** critical ADRs accepted; data ownership, module interfaces and threat model reviewed.

**V1:** traceability complete, blocking failures resolved, physical-device alarm claims validated, license and third-party assets reviewed, release build reproducible.

## 12. Stop-and-escalate conditions

Stop and report before proceeding when:
- a requested feature requires private APIs, DRM bypass, hidden permissions or unsupported background execution;
- an alarm reliability claim cannot be substantiated;
- the proposed minimum OS must change;
- a migration may destroy user data;
- a dependency introduces unreviewed licensing/security implications;
- required physical-device evidence is unavailable but release acceptance is requested;
- repository contents conflict with this document or accepted ADRs.

Do not solve these by quietly reducing requirements or claiming success.

---

**Agent reminder:** The project's value is a trustworthy, sustainable clock and media experience on older devices. Correctness and user control matter more than feature count.
