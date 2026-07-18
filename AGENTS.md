# mixi-bro repository instructions

These instructions apply to the entire repository. A deeper `AGENTS.md` or `AGENTS.override.md` may add or override rules for its subtree, but it must not weaken the security, licensing, privacy, accessibility, or release requirements below.

## Mission

Build **mixi-bro**, a remote-first Android browser inspired by and derived from TV Bro, with:

- GeckoView as the only production web engine;
- a Material Design 3 Expressive interface;
- first-class Android TV / Google TV D-pad, keyboard, mouse, and voice input;
- an adaptive touch experience for phones and tablets where practical;
- safe WebExtension installation and management;
- strong privacy, predictable session restoration, and reliable media playback.

The detailed roadmap and acceptance criteria live in `docs/IMPLEMENTATION_PLAN.md`.

## Product decisions that are not optional

1. The product name is **mixi-bro**. Do not ship the string `TV Bro` as the app name, launcher label, icon text, package name, or application ID.
2. Use the proposed namespace and application ID `io.github.codexclamps.mixibro` unless an ADR explicitly changes it before the first signed release.
3. GeckoView is the sole production engine. Do not add an Android `WebView`, Chromium/Blink, Custom Tabs, or a `geckoExcluded` fallback flavor to render normal browser tabs.
4. Use one application-scoped `GeckoRuntime` and one `GeckoSession` per live tab. Keep Gecko types behind an engine boundary so feature and UI modules do not depend directly on GeckoView APIs.
5. Production builds track the latest published **stable** GeckoView train that has passed the repository compatibility suite. Pin the exact artifact version in the version catalog. Beta and Nightly may exist only in clearly labeled developer builds.
6. The minimum supported Android version is API 26 unless a verified GeckoView requirement forces it higher. Compile and target the current production Android SDK after CI validation.
7. Use Kotlin, Gradle Kotlin DSL, Java 17, coroutines/Flow, Jetpack Compose, Material 3, and TV Material components. Avoid XML UI except for platform-required resources or a documented interop reason.
8. The browser must remain usable with only a D-pad and Back button. Every action reachable by touch must have an obvious focus path and keyboard/remote equivalent.
9. Extension support must never silently grant permissions, bypass signing policy, expose unrestricted native messaging, or enable an extension in private browsing without explicit user consent.
10. Never commit signing keys, API keys, tokens, telemetry secrets, private certificates, downloaded proprietary test media, or user browsing data.

## Upstream source and licensing

TV Bro's license permits modified source and binary redistribution subject to conditions. Every contributor must preserve those obligations.

- Retain the upstream copyright and license text in the repository and source distributions.
- Use a new app name, icon, package name, application ID, signing identity, and release artwork.
- The in-app About screen must clearly state that mixi-bro uses TV Bro source code, credit Fedir Tsapana, and link to `https://github.com/truefedex/tv-bro`.
- Keep a machine-readable third-party notices file and show it from About > Open-source licenses.
- When copying or adapting an upstream file, retain its applicable header and record meaningful modifications in Git history.
- Do not imply affiliation with or endorsement by TV Bro, Mozilla, Firefox, or Google.
- Do not use Firefox trademarks, logos, or branded artwork. GeckoView is the engine technology, not the product identity.

If a change creates licensing uncertainty, stop that workstream and assign `architecture_explorer` to inventory provenance before implementation continues.

## Intended module boundaries

Use this target shape unless an accepted ADR documents a better boundary:

```text
:app                         application, dependency wiring, top-level navigation
:design-system               mixi-bro Material 3 Expressive tokens and adaptive components
:core:model                  immutable domain models and value types
:core:common                 dispatchers, result types, logging interfaces, small utilities
:core:data                   Room/DataStore repositories, migrations, import/export
:core:browser                engine-neutral browser, tab, profile, permission, and download ports
:engine:gecko                GeckoRuntime, GeckoSession, GeckoView, delegates, extension adapter
:feature:browser             browser chrome and page host
:feature:home                new-tab/home experience and shortcuts
:feature:tabs                tab switcher, groups if later approved, restore UX
:feature:bookmarks           bookmark folders, edit, import/export
:feature:history             history search, retention, clear-data controls
:feature:downloads           download list, progress, retry, open/share controls
:feature:extensions          catalog/install flows, permission prompts, manager, action UI
:feature:settings            settings, privacy, site permissions, accessibility
:feature:about               attribution, engine/build details, licenses
:benchmark                   startup, scrolling, tab-switching, and memory benchmarks
```

Dependency direction is feature/UI -> domain ports -> data/engine implementations. `:engine:gecko` may depend on `:core:browser`; `:core:browser` must not depend on GeckoView. Do not create a generic abstraction merely to hide a single line of code; boundaries should protect lifecycle, testability, or policy.

## GeckoView engineering rules

- Create `GeckoRuntime` once from the application process and make initialization idempotent.
- Build runtime settings in one audited factory. Separate release-safe settings from developer-only remote debugging, console logging, and `about:config` access.
- Give each tab an engine-neutral ID. Map it to at most one open `GeckoSession`; suspend or close background sessions under memory pressure while retaining restorable state.
- Attach a session to only one visible `GeckoView` at a time. Treat Compose's `AndroidView` as a host, not as the owner of browser state.
- Model navigation, progress, title, icon, security information, find-in-page, fullscreen, media, prompts, permissions, content blocking, downloads, and crashes as typed events/state.
- Never execute arbitrary JavaScript by constructing a `javascript:` URL. Use a reviewed built-in WebExtension or a narrowly scoped messaging mechanism.
- Keep private profiles isolated. Private tabs must not write history, thumbnails, form state, or extension data into the normal profile.
- Persist only the minimum session data needed for restoration. Encrypt or avoid sensitive data; never log visited URLs in release builds.
- Handle renderer/content-process crashes without crashing the app. Offer reload and restore, and record privacy-safe diagnostics.
- Treat certificate errors, mixed content, authentication prompts, permission requests, external intents, file pickers, and downloads as security boundaries with explicit user decisions.
- Validate media fullscreen, Picture-in-Picture where supported, DRM/Widevine behavior, remote playback controls, and audio focus on real TV hardware before release.
- The engine version is part of the security surface. A stable GeckoView update that contains security fixes takes precedence over feature work, but still runs the compatibility suite.

## WebExtension rules

Implement two distinct extension classes:

1. **Built-in bridge extension**: packaged in app assets, versioned with the app, and limited to capabilities mixi-bro itself requires, such as safe content-script coordination and browser-action plumbing.
2. **User extensions**: installed and managed through GeckoView's `WebExtensionController`, with visible source, identity, version, permission, update, enable/disable, private-mode, and uninstall controls.

Required safeguards:

- Accept production user extensions only from an approved signed source, initially Mozilla Add-ons or a validated signed XPI flow. Local unsigned installation belongs behind a developer-mode gate in non-release builds.
- Show requested host and API permissions before installation and material permission changes. Never pre-check optional permissions.
- Maintain an explicit native-messaging allowlist. User extensions do not get native messaging by default.
- Validate extension IDs, URLs, redirects, MIME types, package size, signatures, and update metadata.
- Surface unsupported or partially supported APIs in a compatibility view rather than failing silently.
- Browser-action and page-action UI must be remote-focusable. Popups/options pages need lifecycle limits and safe dismissal.
- Enable private-browsing access per extension only after separate consent; default is disabled.
- Do not build a general-purpose script injection UI in the first release.
- Assign `extension_security_reviewer` before merging any change to install, update, permissions, native messaging, content scripts, or extension storage.

## Material 3 Expressive and TV UX rules

- Centralize color, type, shape, spacing, elevation, motion, focus, and state tokens in `:design-system`.
- Prefer stable Compose Material 3 and TV Material releases. Experimental expressive APIs may be isolated behind wrappers and an opt-in annotation; do not spread experimental APIs through feature modules.
- Preserve a coherent visual identity rather than copying Firefox or TV Bro artwork.
- Use expressive shape and motion to communicate hierarchy and state, not as decoration that delays browsing.
- Respect reduced-motion settings. Avoid continuous ambient animation on resource-constrained TVs.
- Provide large focus targets, visible focus rings/glows, strong contrast, deterministic focus restoration, and sensible D-pad traversal.
- Keep critical browser chrome inside overscan-safe areas. Test 720p, 1080p, and 4K density/scaling.
- Do not mix phone Material components and TV Material components in the same focus hierarchy without an adapter that has been tested with TalkBack and D-pad navigation.
- Support font scaling, TalkBack, switch access where feasible, high contrast, RTL, long translations, keyboard shortcuts, and screen-reader labels.
- Destructive actions require confirmation or an undo path. Permission and certificate prompts must never be visually ambiguous.
- Assign `material3_tv_designer` for any new screen, navigation model, focus change, motion pattern, or design-token change.

## Data, privacy, and observability

- Use Room for relational browsing data and DataStore for small typed settings. All schema changes require migrations and migration tests.
- Store bookmarks, history, downloads, tab metadata, site permissions, and extension records behind separate repository interfaces.
- Private browsing writes no normal history. Clearing data must map clearly to Gecko storage categories and local database categories.
- Logging is structured and redacted. Release logs must not contain full URLs, page titles, form values, query terms, cookies, authorization headers, extension messages, or local file paths.
- Telemetry is off by default unless the project later adopts a documented, consent-based policy. No third-party analytics SDK may be added without an ADR and privacy review.
- Crash reports must be opt-in or use platform mechanisms with documented redaction and retention.

## Multi-agent operating model

Project-scoped Codex roles are defined in `.codex/agents/*.toml`; concurrency limits are in `.codex/config.toml`.

### Available roles

- `architecture_explorer`: read-only repository mapping, upstream comparison, dependency and provenance analysis.
- `geckoview_specialist`: read-only GeckoView lifecycle/API research, delegate and compatibility review.
- `material3_tv_designer`: read-only Compose, Material 3 Expressive, accessibility, and remote-focus review.
- `extension_security_reviewer`: read-only WebExtension threat review and permission/native-messaging audit.
- `test_release_engineer`: CI, test matrix, reproducibility, performance, packaging, and release checks.
- `implementer`: bounded code changes after the lead agent has synthesized a design.

### Required orchestration for non-trivial work

A task is non-trivial when it crosses a module boundary, changes engine/session lifecycle, adds a screen, changes persistence, touches extension behavior, modifies security/privacy policy, or is likely to change more than three production files.

For such work, the lead agent must:

1. Read this file and the relevant plan/ADR documents.
2. Spawn independent read-only agents in parallel for separable questions. Use at least two relevant specialist roles; engine-plus-UI work normally uses three.
3. Give every subagent a narrow question, explicit file scope, expected output, and a requirement to cite paths/symbols or official sources.
4. Wait for all assigned investigations, reconcile disagreements, and write one implementation decision before assigning edits.
5. Give write ownership to one `implementer` per file or module. Do not allow concurrent agents to edit the same files.
6. Run focused checks, then the repository quality gate.
7. Ask the relevant read-only specialist to review the diff. Security-sensitive extension work always includes `extension_security_reviewer`; engine work includes `geckoview_specialist`; UI work includes `material3_tv_designer`.
8. Summarize findings, decisions, tests, residual risks, and follow-up items in the PR.

Parallel agents are encouraged for exploration, API verification, test design, and review. Serialize migrations, version-catalog edits, shared design tokens, Gradle settings, and release configuration unless worktrees and ownership are clearly isolated.

For a small typo, localized test fix, or single-file documentation change, do not spawn agents merely to satisfy a number.

## Planning and change workflow

1. Restate the user-visible behavior and acceptance criteria.
2. Inspect current code and tests before proposing abstractions.
3. Check the relevant milestone in `docs/IMPLEMENTATION_PLAN.md`.
4. For a durable architectural choice, create or update an ADR under `docs/adr/`.
5. Make the smallest coherent change that moves one acceptance criterion to done.
6. Add tests with the implementation; do not defer all tests to a later phase.
7. Run formatting, static analysis, unit tests, and the narrowest applicable Android/instrumentation checks.
8. Keep commits focused. Use a feature branch and draft PR for normal work after repository bootstrap.
9. Update the plan when scope, dependencies, compatibility, or risks materially change.

Do not claim a command passed unless its output was observed. If the environment lacks Android SDK, emulator, credentials, network access, or hardware, record the exact unrun checks and why.

## Expected commands after bootstrap

Use the Gradle wrapper committed by the project. The definitive task names may evolve; update this section when they do.

```bash
./gradlew spotlessCheck lint test
./gradlew :app:assembleDebug
./gradlew :app:assembleRelease
./gradlew connectedCheck                 # emulator/device available
./gradlew :benchmark:connectedCheck      # benchmark-capable device available
```

Useful focused checks should include module tests, Room migration tests, Compose UI tests, engine instrumentation tests, and extension fixture tests. Never disable a failing check to make CI green without documenting and fixing the underlying issue.

## Testing expectations

- Domain reducers/state machines: deterministic unit tests.
- Persistence: DAO, repository, import/export, and every Room migration path.
- Gecko adapter: instrumentation tests using local deterministic HTTP fixtures; do not depend on public websites in gating CI.
- UI: Compose semantics, keyboard/D-pad traversal, focus restoration, TalkBack labels, Back behavior, and screenshots where stable.
- Extensions: signed fixture install, permission acceptance/denial, enable/disable, update, private access, action popup, incompatible API, uninstall, and malicious package cases.
- Browser essentials: navigation, redirects, downloads, uploads, authentication, permissions, fullscreen video, media controls, process death, crash recovery, offline/error pages, and external-intent handling.
- Performance: startup, first content paint proxy, tab switch, memory under multiple tabs, frame timing, and APK/AAB size budgets.
- Release: reproducible unsigned artifacts, signing only in protected CI, SBOM/notices generation, dependency and secret scans, and ABI/device matrix smoke tests.

## Definition of done

A change is done only when:

- the requested behavior and failure states are implemented;
- affected code compiles and relevant tests pass;
- no WebView/Blink fallback or hidden network dependency was introduced;
- TV remote, keyboard, mouse, touch, accessibility, and Back behavior were considered and tested as applicable;
- privacy, extension permissions, external intents, and data retention were reviewed where applicable;
- localization-ready strings replace user-visible literals;
- new APIs and non-obvious lifecycle decisions are documented;
- license and attribution obligations remain satisfied;
- dependency/version changes are pinned and recorded;
- the PR states validation performed and any residual limitations honestly.

## Review priorities

Review in this order:

1. security, privacy, certificate/permission behavior, and extension trust;
2. data loss, session corruption, and process-death recovery;
3. GeckoRuntime/GeckoSession lifecycle and memory safety;
4. remote focus, accessibility, and navigation correctness;
5. correctness, tests, maintainability, and performance;
6. visual polish.

Never approve a visually polished browser that weakens trust boundaries or loses user data.
