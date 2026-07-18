# mixi-bro implementation plan

**Status:** proposed implementation baseline  
**Verified:** 2026-07-18  
**Target repository:** `codex-clamps/mixi-bro`  
**Reference project:** `truefedex/tv-bro`, release 2.1.6 / commit `10b8b1a9ff29fcbe5c55052eec61caf553d6964f`

## 1. Executive recommendation

Build mixi-bro as a controlled, license-compliant modernization of TV Bro rather than a superficial rename or an unrelated greenfield browser.

The recommended sequence is:

1. Preserve the TV Bro source history or import it at a clearly recorded commit.
2. Immediately establish the new identity: **mixi-bro**, a new icon, new application ID, new package namespace, new signing key, and the required TV Bro attribution in About.
3. Make GeckoView the only production engine. Remove the WebView/Blink implementation and the `geckoExcluded` product flavor instead of maintaining two engines.
4. Keep the useful engine-neutral behavior from TV Bro, but replace fragile or over-broad abstractions with a typed browser core and a dedicated `:engine:gecko` adapter.
5. Rebuild the user interface in Jetpack Compose using a mixi-bro Material Design 3 Expressive design system and stable TV Material components.
6. Ship extension support in two stages: a tightly controlled built-in bridge extension, followed by signed user-installed WebExtensions with explicit permission and private-mode controls.
7. Treat D-pad focus, session restoration, memory pressure, certificate/permission prompts, and extension trust as release-blocking behavior, not polish.

This path retains TV Bro's useful remote-browser feature set while avoiding the two largest long-term costs in the current codebase: a dual-engine architecture and a legacy view-based UI.

## 2. Facts that shape the plan

### 2.1 TV Bro baseline

The inspected TV Bro version already contains more than the public README implies:

- The public feature set includes remote control input, tabs, bookmarks, voice search, user-agent switching, downloads, history, and shortcuts.
- The repository has `:app`, `:app:common`, and `:app:gecko` modules.
- The app still has `geckoIncluded` and `geckoExcluded` engine flavors, so Gecko is optional rather than foundational.
- The Gecko implementation creates one shared `GeckoRuntime`, one `GeckoSession` per tab, delegate objects for navigation/progress/prompts/permissions/media/history/content blocking, and a built-in WebExtension bridge.
- The inspected version catalog pins GeckoView `147.0.20260212191108`.
- The application uses XML/ViewBinding/AppCompat rather than Compose.
- The current app ID and namespace are `com.phlox.tvwebbrowser`.
- TV Bro's modified-binary license requires a new name, icon, and application ID and requires an About view crediting TV Bro with a link to its source repository.

Therefore, this is not a basic “replace Android WebView with GeckoView” project. The engine proof of concept exists; the real work is to make Gecko the sole, robust foundation and modernize the architecture and experience around it.

### 2.2 Current platform baseline on 2026-07-18

- Firefox 152 is the stable Firefox train; Firefox 153 is still Beta and is scheduled for 2026-07-21. Production should therefore begin on the latest published GeckoView **152 stable** artifact available from Mozilla Maven, with its exact timestamped version pinned after metadata resolution.
- GeckoView is self-contained, is intended for browser embedding, requires Java 17 compatibility, and exposes `GeckoRuntime`, `GeckoSession`, `GeckoView`, delegates, storage/content blocking, and `WebExtensionController` APIs.
- GeckoView user-extension installation validates Mozilla-signed extensions and provides an install-prompt callback for permissions. Built-in extensions are packaged under `resource://android/assets/...`, are not signed, and can use native messaging; they therefore require a stricter app-controlled trust boundary.
- Stable Compose Material 3 is 1.4.0; stable core Compose groups are 1.11.4. Stable TV Material is 1.1.0 and TV Foundation is 1.0.0.
- Material 3 Expressive is part of Android's Material 3 direction, but some expressive APIs remain experimental. The production design system should use stable APIs by default and contain any experimental opt-ins behind local wrappers.
- Codex supports project-scoped custom agents under `.codex/agents/*.toml`, with global project limits under `.codex/config.toml`. Root instructions in `AGENTS.md` can require delegation for suitable work.

These numbers are a dated bootstrap baseline, not a permanent promise. At implementation time, resolve and pin the newest stable security-supported versions, run the compatibility suite, and document every deviation.

## 3. Product definition

### 3.1 Product statement

mixi-bro is a private, remote-friendly Gecko browser for Android TV and Google TV that also adapts to keyboard, mouse, and touch devices. It should feel deliberately designed for a ten-foot interface rather than like a phone browser enlarged onto a television.

### 3.2 Primary users

1. **TV remote user:** browses streaming, news, documentation, or local services using only D-pad, Select, Back, and voice input.
2. **Power TV user:** uses a Bluetooth keyboard, air mouse, gamepad, or physical mouse and needs predictable shortcuts and cursor behavior.
3. **Privacy-conscious user:** expects content blocking, clear site permissions, private tabs, and no surprise telemetry.
4. **Extension user:** installs a small, compatible set of signed WebExtensions and needs transparent permissions and reliable action/popup behavior.
5. **Touch user:** runs the same APK on a phone or tablet and expects adaptive browser chrome without weakening the TV experience.

### 3.3 Product principles

- **Remote first, not remote only.** D-pad behavior defines the minimum experience; touch and mouse enhance it.
- **Trust is visible.** Certificate state, permissions, extension permissions, external-app launches, and data-clearing effects must be understandable.
- **Fast path to content.** Expressive UI should clarify hierarchy and focus, never obstruct page loading.
- **State survives reality.** Process death, device sleep, crashes, low memory, rotation, and TV launcher interruptions must not casually lose tabs.
- **Stable engine over novelty.** Stable GeckoView is the release channel. Beta/Nightly builds are opt-in engineering tools.
- **No hidden second browser.** Normal tabs always use GeckoView.

### 3.4 Version-one scope

Version one includes:

- normal and private browsing;
- omnibox search/navigation;
- multiple tabs with restore;
- bookmarks and folders;
- history and clear-data controls;
- home/new-tab shortcuts;
- download management;
- page upload/file picker;
- find in page;
- desktop/mobile user-agent modes plus per-site override if proven necessary;
- zoom and text scaling;
- fullscreen video, audio focus, media controls, and Picture-in-Picture where supported;
- content blocking and per-site exceptions;
- site permissions and certificate/security information;
- voice search when platform support is available;
- D-pad virtual cursor mode for pages that are not spatial-navigation friendly;
- signed WebExtension install, permission prompt, list, enable/disable, uninstall, update handling, action popup, options page, and private-access toggle;
- import/export of bookmarks and user settings in a documented format;
- About, licenses, engine/build information, and TV Bro attribution;
- release artifacts for at least arm64-v8a and x86_64 testing, with ABI policy decided before store release.

### 3.5 Explicit non-goals for version one

- Firefox Sync or Mozilla account integration;
- cross-device cloud sync;
- a general userscript engine;
- arbitrary unsigned production extensions;
- desktop-class developer tools inside the app;
- password-manager or credit-card storage beyond safe Android/Gecko platform integration;
- VPN, proxy subscription, Tor, or IP-protection services;
- tab groups unless basic tabs are stable and the product owner accepts the extra TV navigation complexity;
- a custom rendering engine;
- Chrome-extension compatibility promises;
- Android versions below API 26;
- exact visual parity with Firefox or TV Bro.

## 4. Legal, naming, and provenance plan

Before application code changes begin:

1. Add TV Bro's `LICENSE.md` unchanged to the root or `licenses/TV_BRO_LICENSE.md` and retain copyright.
2. Add `NOTICE.md` describing:
   - that mixi-bro derives from TV Bro;
   - the pinned upstream commit used for the initial import;
   - the upstream URL;
   - major architectural modifications;
   - Mozilla/GeckoView and AndroidX attribution without implying endorsement.
3. Put the required attribution in `:feature:about` and make it reachable with a remote in no more than three actions from Settings.
4. Replace all product identifiers:
   - app name: `mixi-bro`;
   - namespace/application ID: proposed `io.github.codexclamps.mixibro`;
   - deep-link authorities;
   - FileProvider authority;
   - update metadata URLs;
   - extension IDs/native-app names;
   - launcher shortcuts;
   - notification channel IDs where identity matters;
   - database and preference names where migration from TV Bro is not intended.
5. Design an original icon and brand system. Do not recolor or slightly alter TV Bro or Firefox artwork.
6. Decide whether existing TV Bro installations should migrate. The default recommendation is **no in-place migration**, because the application ID and signing identity must differ. Offer explicit bookmark export/import instead.
7. Preserve upstream history where practical:
   - add `truefedex/tv-bro` as `upstream`;
   - create an immutable tag such as `upstream-tv-bro-2.1.6` at the imported commit;
   - make mixi-bro changes in normal commits after that point;
   - never rewrite the attribution-bearing history after public release.

**Exit criterion:** a legal/provenance checklist is reviewed before any APK is distributed outside the development team.

## 5. Technology and dependency baseline

### 5.1 Recommended build baseline

Start from the current TV Bro toolchain only after confirming compatibility:

- Gradle wrapper compatible with Android Gradle Plugin 9.2.x;
- Android Gradle Plugin 9.2.1 as the initial candidate;
- Kotlin 2.3.21 as the initial candidate;
- Java 17 toolchain and source/target compatibility;
- compileSdk 36 and targetSdk 36 initially;
- minSdk 26;
- KSP for Room processors;
- a version catalog at `gradle/libs.versions.toml`;
- reproducible dependency locking or verification metadata;
- Compose compiler matching the Kotlin plugin's supported integration;
- stable Compose Material 3 1.4.0, Compose 1.11.4, TV Material 1.1.0, TV Foundation 1.0.0 as dated initial candidates;
- latest stable Room, DataStore, Lifecycle, Navigation Compose, Activity Compose, Benchmark, and AndroidX Test versions that pass CI;
- latest stable GeckoView 152 build from `https://maven.mozilla.org/maven2/`, pinned exactly.

Do not write `+`, `latest.release`, or an unbounded Maven range. Create a scheduled dependency-update workflow that opens reviewable PRs; do not update GeckoView directly on `main`.

### 5.2 Release channels

Use app build types or product flavors only for real policy differences:

- `debug`: stable GeckoView, debugging enabled, local test hooks;
- `nightly`: GeckoView Nightly or Beta, visible warning, separate app ID suffix, no production guarantees;
- `release`: stable GeckoView, debugging/about:config/console output disabled, minification and resource shrinking enabled;
- optional store dimension only when distribution policy genuinely differs, not for engine selection.

Remove `geckoIncluded` and `geckoExcluded`. A browser whose identity is Gecko must not compile without its engine.

### 5.3 Direct GeckoView versus Android Components

**Recommended default:** use direct GeckoView behind mixi-bro's `BrowserEngine` port.

Reasons:

- TV Bro already contains a functioning direct GeckoView implementation and delegate model.
- The product needs a highly custom TV-first Compose experience rather than a Fenix-like frontend.
- Direct use keeps extension permissions, runtime settings, and tab lifecycle explicit.
- It avoids importing a broad set of Android Components before their concrete value is known.

Create a short technical spike before finalizing ADR-0001:

- implement one tab, navigation, state restore, permission prompt, and one signed extension with direct GeckoView;
- estimate equivalent work with `browser-engine-gecko` and relevant Android Components;
- compare APK size, API surface, extension plumbing, maintenance burden, testability, and update cadence;
- adopt Android Components only if it measurably removes high-risk browser infrastructure without dictating an unsuitable UI architecture.

This is a decision gate, not permission to keep two engines indefinitely.

## 6. Target architecture

### 6.1 High-level flow

```text
Compose / TV Material UI
        |
        v
BrowserCoordinator + feature ViewModels
        |
        v
:core:browser ports and state machines
        |                       \
        v                        v
:engine:gecko                :core:data
GeckoRuntime/Session/View     Room/DataStore/files
        |
        v
Gecko content processes + WebExtensions
```

The UI observes immutable app state and sends intents. It does not directly call `GeckoSession`. The engine adapter translates Gecko callbacks into typed events and executes commands from the browser coordinator.

### 6.2 Module plan

#### `:app`

Responsibilities:

- `Application` class and dependency graph;
- top-level activity, navigation, device/form-factor detection;
- process lifecycle and startup sequencing;
- release channel wiring;
- notifications, app shortcuts, intent entry points;
- no detailed feature implementation.

#### `:design-system`

Responsibilities:

- mixi-bro color, typography, shape, spacing, elevation, motion, and focus tokens;
- stable wrappers around Material 3 and TV Material;
- adaptive scaffolds and browser chrome primitives;
- focus-ring, selected, pressed, loading, disabled, and danger states;
- preview catalog and screenshot fixtures;
- no dependency on GeckoView or repositories.

#### `:core:model`

Immutable models:

- `ProfileId`, `TabId`, `WindowId`, `ExtensionId`, `DownloadId` value classes;
- `BrowserTab`, `TabSnapshot`, `NavigationEntry`, `PageSecurityState`;
- `SitePermission`, `PermissionDecision`, `ContentBlockingState`;
- `Bookmark`, `BookmarkFolder`, `HistoryEntry`, `DownloadRecord`;
- `ExtensionMetadata`, `ExtensionPermission`, `ExtensionState`;
- error/result types with no Android UI dependency.

#### `:core:browser`

Ports and deterministic state:

- `BrowserEngine`, `EngineRuntime`, `EngineSession`, `EngineViewHost`;
- `BrowserStore`/reducer or coordinator state machine;
- tab creation, selection, closing, suspension, restoration;
- navigation command normalization;
- prompt, permission, download, media, and extension policy interfaces;
- test fakes that need no Android emulator.

Avoid a one-to-one wrapper for every Gecko API. Expose product concepts, not engine implementation details.

#### `:engine:gecko`

Responsibilities:

- runtime factory and singleton ownership;
- GeckoSession creation/settings and delegate registration;
- Compose `AndroidView` host integration;
- session state serialization/restoration;
- navigation, progress, content, history, permission, prompt, media, selection, content-blocking, and crash delegates;
- extension controller and built-in bridge installation;
- Gecko storage and clear-data mapping;
- version/build diagnostics;
- instrumentation fixtures and compatibility tests.

No Room DAOs or feature UI should live here.

#### `:core:data`

Responsibilities:

- Room database, DAOs, schema and migrations;
- DataStore settings and strongly typed serializers;
- repositories for bookmarks, history, tabs, downloads, site permissions, extension records, and settings;
- import/export and backup validation;
- retention/clear-data transactions;
- encryption strategy for sensitive records, if any.

#### Feature modules

Create feature modules only when the screen or workflow becomes active. Each owns navigation entry points, ViewModels, UI state, and Compose screens, while depending on core ports/repositories rather than implementations.

### 6.3 State ownership

Use a single source of truth for browser tabs:

```text
BrowserState
- activeProfile
- selectedTabId
- orderedTabIds
- tabSummaries[TabId]
- overlay/prompt state
- fullscreen/media state
- engineHealth state
```

A live tab has:

```text
TabRecord (persistent)
+ EngineSessionHandle (runtime only)
+ ViewAttachment (visible only)
```

Rules:

- The selected tab gets the visible `GeckoView` attachment.
- Background sessions may stay open up to a memory budget.
- Older background tabs are suspended: flush session state, detach view, close session if needed, retain restorable metadata.
- A private tab never persists normal history or thumbnails and is discarded when the private profile closes.
- Process death restoration reconstructs records first and opens Gecko sessions lazily.
- A close action is transactional: update active selection, persist order, close session, remove snapshot, and offer undo where safe.

### 6.4 Structured concurrency

- Create one application `CoroutineScope` with supervised lifecycle-owned work only where truly process-wide.
- Every tab/session has a cancellable scope tied to its handle.
- Convert Gecko `GeckoResult` callbacks with small, tested adapters; preserve cancellation behavior and thread requirements.
- UI ViewModels own screen work and never leak Activity/GeckoView references.
- Use `StateFlow` for observable state and `SharedFlow` or channels only for events that cannot be represented as state.
- Define dispatchers behind interfaces for deterministic tests.

## 7. GeckoView implementation design

### 7.1 Runtime initialization

Create `GeckoRuntimeProvider` with an idempotent `get()`/`initialize()` contract.

Release settings should include only reviewed values:

- production-safe content blocking defaults;
- preferred color scheme from settings/system;
- font scaling/accessibility mapping;
- locale mapping;
- crash handling policy;
- extension process setting after compatibility validation;
- no remote debugging;
- no `about:config` unless explicitly gated in developer builds;
- no web console output in release;
- no blanket insecure-connection allowance.

Developer builds may enable remote debugging and diagnostics through a visible developer setting. Never infer developer mode merely from a hidden URL.

Prewarm the runtime only after measuring startup and memory tradeoffs. A startup initializer that creates Gecko too early may increase launcher latency and resident memory on low-end TVs.

### 7.2 Session factory

`GeckoSessionFactory` receives a product-level `SessionSpec`:

- normal/private profile;
- mobile/desktop viewport and user-agent mode;
- tracking-protection state;
- initial restore state or URL;
- accessibility/text scale;
- extension private-access policy.

It creates one session, installs all delegates before opening it, opens against the runtime, and returns an engine-neutral handle.

### 7.3 Delegate map

Implement and test separate adapters:

- `NavigationDelegateAdapter`: location, redirects, external intents, new-window requests, back/forward availability, load errors, certificate state;
- `ProgressDelegateAdapter`: progress, title, icon, security changes, session-state updates, crash/recovery signal;
- `ContentDelegateAdapter`: fullscreen, context menus, close requests, first-contentful paint proxy if exposed;
- `PromptDelegateAdapter`: alert/confirm/text/auth/file/color/date prompts with queueing and cancellation;
- `PermissionDelegateAdapter`: Android runtime permissions plus web-content permission, site persistence, one-time decisions;
- `HistoryDelegateAdapter`: visit/title events with private-mode filtering and debounce/deduplication;
- `ContentBlockingDelegateAdapter`: blocked event count/log and per-site exception state;
- `MediaSessionDelegateAdapter`: playback metadata, transport commands, audio focus, notification/TV media session;
- `SelectionActionDelegateAdapter`: copy/share/search/open actions appropriate to remote and touch;
- `DownloadDelegateAdapter`: response/download request into Android storage workflow;
- crash/console delegates enabled only by release policy.

Every adapter converts callbacks into typed events and has unit tests around mapping. Instrumentation tests validate thread/lifecycle behavior.

### 7.4 Compose view hosting

Use a `GeckoViewHost` composable backed by `AndroidView`, but keep session ownership outside composition.

Requirements:

- attach the selected session exactly once;
- detach before reusing or destroying a view;
- preserve focus transfer between browser chrome and web content;
- map Back first to overlays/prompts, then page history, then tab/app behavior;
- support pointer capture and a virtual cursor overlay without intercepting normal Gecko input unexpectedly;
- handle window insets, IME resizing, fullscreen transitions, and configuration changes;
- avoid recreating the view/session on unrelated recompositions;
- explicitly test Activity recreation and tab switching under rapid remote input.

### 7.5 Navigation and URL handling

Create one `OmniboxParser`:

- trim and normalize input;
- distinguish a valid absolute URI, host-like input, internal page, and search query;
- default unknown schemes to search, not execution;
- allow only an explicit set of externally launchable schemes;
- canonicalize IDNs for display while evaluating security on the actual URI;
- expose a clear confirmation for intent/app launches;
- prevent `javascript:`, `data:`, local file, privileged Gecko, and extension URLs from being launched through untrusted UI paths;
- support QR/voice/clipboard inputs through the same parser.

### 7.6 Session restoration

Persist:

- tab ID/order/title/last URL/profile;
- serialized Gecko session state when available;
- selected tab;
- thumbnail only for normal tabs and only under a size/retention budget;
- whether a tab was playing media or fullscreen only as diagnostic state, not auto-resume permission.

Restoration algorithm:

1. Read tab records without starting all sessions.
2. Select the previous active normal tab, or a fresh home tab if invalid.
3. Start Gecko runtime.
4. Create/restore only the selected session.
5. Lazily restore another session when selected or prewarmed under a memory budget.
6. If state decode fails, load the last safe URL and mark the snapshot invalid.
7. Never restore private tabs after process restart unless a deliberate product decision and safe profile isolation are implemented.

### 7.7 Memory policy

TV devices vary widely. Create measurable levels rather than an arbitrary tab cap:

- active: selected session attached to the view;
- warm: open background session, no view;
- suspended: serialized state, session closed;
- discarded: metadata retained, reload last URL on selection.

Use `onTrimMemory`, process importance, recent-use order, media activity, form-data signals when available, and device memory class. Never suspend a tab with an unresolved permission/prompt or active download without a defined handoff.

### 7.8 Internal pages

Prefer app-native Compose screens for home, settings, history, downloads, extensions, and errors. Use internal Gecko pages only where Gecko owns the capability and a native replacement would create risk. Do not expose unrestricted `about:` navigation in release.

If a built-in extension injects an app home page, define why it is superior to a native screen and security-review all messages. The default recommendation is a native Compose new-tab screen.

## 8. WebExtension subsystem

### 8.1 Capability levels

#### Level A: built-in bridge

Package under `app/src/main/assets/extensions/mixi_bridge/` with a stable extension ID such as `mixi-bridge@codexclamps.invalid` until a real controlled domain/ID policy is chosen.

Allowed responsibilities:

- content-script communication required for virtual cursor or page metadata;
- browser-action/page-action event forwarding;
- narrowly scoped page behavior not exposed by GeckoView;
- local deterministic test hooks in debug builds.

Rules:

- minimum permissions;
- version synchronized with app build;
- JSON-schema or sealed-message validation;
- explicit native app/channel names;
- no arbitrary code or script string from Android;
- no network calls unless documented and user-visible;
- `ensureBuiltIn()` for upgrades, with failure surfaced in diagnostics;
- extension unavailable must degrade features safely, not crash browsing.

#### Level B: signed user extensions

Initial sources:

- curated Mozilla Add-ons collection/API if product/legal review approves;
- direct HTTPS link to a Mozilla-signed XPI after validation and confirmation;
- optional local signed XPI picker.

Do not promise that every desktop Firefox add-on works. Maintain a compatibility matrix and curated recommendations based on tested APIs.

### 8.2 Extension data model

Persist product-level metadata, not entire packages:

```text
ExtensionRecord
- id
- name
- version
- sourceUri/sourceType
- enabled
- allowedInPrivate
- requiredPermissions
- optionalPermissionsGranted
- hostPermissions
- dataCollectionPermissions
- updateState
- compatibilityState
- lastErrorCode
```

Treat GeckoView's installed-extension list as authoritative for runtime state and reconcile it against local display metadata at startup.

### 8.3 Install flow

1. User selects an extension from an approved source or enters a permitted HTTPS XPI URL.
2. Validate URL scheme, host policy, redirect chain, size ceiling, content type, and source context.
3. Call GeckoView install using a meaningful installation method identifier.
4. Receive metadata and permission arrays in the prompt delegate.
5. Show a full-screen, remote-focusable install review:
   - extension identity/version/source;
   - required API permissions;
   - host permissions grouped and humanized;
   - data-collection permissions if exposed;
   - private browsing default disabled;
   - risk explanation for broad access.
6. User accepts or denies. Back equals deny.
7. On success, reconcile the installed list and show action placement/compatibility status.
8. On error, map Gecko install codes to actionable, non-technical messages while retaining a diagnostic code.

### 8.4 Enable, disable, uninstall, and update

- Use GeckoView controller APIs rather than mutating local state optimistically.
- Disable must close or invalidate affected extension pages/action popups.
- Uninstall confirmation states that extension data will be removed.
- Update permission increases require a new prompt; never silently accept broader hosts or APIs.
- Protect against downgrade and unexpected extension-ID changes.
- Provide recovery for an extension repeatedly crashing its process: auto-disable only under a documented policy and tell the user why.

### 8.5 Actions, popups, and options

- Map action metadata to toolbar/menu entries.
- Popup content renders in a bounded Gecko session/view, not in a raw WebView.
- The popup has deterministic focus entry, close on Back, size limits, and timeout/crash handling.
- Options pages open in a normal internal tab with an extension identity banner.
- Badge text and enabled state update through delegates without leaking cross-tab state.
- Avoid showing every action permanently in compact TV chrome; provide pin/unpin plus an overflow panel.

### 8.6 Private browsing

Default user-extension access is off. A separate details-screen toggle explains that an extension may see private-page content. Persist user consent. Ensure normal and private extension state/storage behavior matches GeckoView guarantees; test it rather than assuming desktop Firefox semantics.

### 8.7 Threat model requirements

At minimum, test:

- malicious redirect to non-HTTPS/local/private-network XPI;
- oversized or wrong-MIME package;
- unsigned/corrupt package;
- ID/version spoofing;
- broad `<all_urls>` permission comprehension;
- permission increase on update;
- native message with wrong extension ID/channel/schema/size;
- extension popup launching external intents;
- private-mode leakage;
- extension storage surviving uninstall unexpectedly;
- crash loop and denial of service;
- malicious page impersonating an extension install UI.

No extension milestone exits without `extension_security_reviewer` findings resolved or explicitly accepted in an ADR.

## 9. Material Design 3 Expressive and TV interaction plan

### 9.1 Visual identity

Create an original system around the idea of “mixing routes/content” without copying other browser brands.

Recommended direction:

- a distinctive rounded-but-not-cartoonish shape family;
- a bold display type style for empty/home states and restrained title styles in browser chrome;
- expressive container shape changes for selected/focused tabs and actions;
- a compact neutral browsing canvas so the page remains dominant;
- focus glow/ring with high contrast independent of color alone;
- dynamic color on compatible touch devices, with a curated TV palette as the default because TV wallpaper color is inconsistent and distance reduces subtle contrast;
- light, dark, and system modes, plus high-contrast overrides.

Create a design-token document before feature screens. Do not let each module choose its own shapes or focus scale.

### 9.2 Adaptive navigation

#### TV/landscape default

- top or side browser chrome inside safe insets;
- omnibox prominent but collapsible during content viewing;
- a tab affordance showing count and active identity;
- overflow panel for bookmarks, history, downloads, extensions, settings;
- optional bottom transient media/navigation strip when useful;
- web content receives focus only after an explicit move/select, so chrome focus does not disappear accidentally.

#### Phone/tablet

- compact bottom or top bar based on width/posture;
- touch gestures and predictive Back integration;
- same domain state and screen modules, different adaptive scaffolds;
- no TV-only focus animation that feels oversized on touch.

### 9.3 Focus model

Document a focus graph for every screen. Core rules:

- opening a screen focuses the primary safe action or first content item, never a destructive action;
- returning restores the previously focused item by stable ID;
- list/grid focus remains visible and scrolls into view;
- side panels trap focus until dismissed, except documented escape paths;
- web content and browser chrome have clear directional boundaries;
- Back closes the deepest overlay/popup/prompt, then exits modes, then page history, then tab/app according to policy;
- long-press and key-repeat behavior is deliberate;
- pointer/mouse hover never steals D-pad focus unexpectedly.

Automate focus traversal tests for high-value screens and maintain manual diagrams for complex browser/extension overlays.

### 9.4 Core screens

#### Onboarding

- privacy summary;
- remote input tutorial;
- default search engine selection if required;
- optional voice permission only at first use, not preemptively;
- extension support explanation without urging broad permissions.

#### Home/new tab

- focused omnibox/search;
- pinned shortcuts with edit/reorder;
- recent normal tabs optionally shown without private history leakage;
- bookmarks entry;
- no sponsored content or network feed in version one.

#### Browser chrome

- Back, Forward, Reload/Stop, Home/New Tab, Omnibox, Tabs, Menu;
- site security/content-blocking indicator;
- extension action overflow;
- progress indication that does not cause layout shift;
- desktop/mobile UA toggle in site controls, not permanently occupying chrome.

#### Tab switcher

- remote-friendly list or grid after usability testing;
- stable tab IDs, title, host, thumbnail, private marker, audio indicator;
- close and undo;
- new tab and new private tab;
- memory/suspended state need not be exposed unless useful.

#### Permission and security prompts

- site identity at top;
- plain-language request and scope;
- Allow once, Allow while using/remember, Deny choices where the API supports them;
- Back denies;
- no focus default on Allow;
- certificate errors use a dedicated danger screen, not a generic dialog.

#### Extensions manager

- installed list with enabled/private/compatibility status;
- details, permissions, source, version, update status;
- install entry and curated catalog only after trust design is complete;
- action pinning;
- clear recovery state for incompatible/crashing extensions.

### 9.5 Motion

Use expressive motion for:

- focus elevation/shape transition;
- tab selection and panel entrance;
- loading-to-content state;
- successful install/bookmark confirmation;
- undo affordance.

Budgets:

- focus feedback must feel immediate;
- navigation motion must not block input;
- reduced-motion mode replaces transforms with fades or no motion;
- avoid heavy blur, continuous gradients, or large compositing surfaces on low-end TVs;
- benchmark representative transitions.

### 9.6 Accessibility and localization

- semantic labels and roles for all icon-only actions;
- state descriptions for selected tab, security state, muted/media state, and extension enabled/private state;
- no information conveyed only by color or animation;
- support 200% font scaling where layouts can reasonably adapt;
- RTL mirroring plus correct Back/Forward semantics;
- externalize every user-visible string from the start;
- test long German-like strings, Thai combining marks, Arabic RTL, CJK, and emoji in titles;
- ensure web content accessibility remains reachable when virtual cursor mode is off;
- do not replace platform accessibility focus with a visual-only custom cursor.

## 10. Feature-by-feature migration matrix

| Capability | TV Bro source treatment | mixi-bro target | Priority |
|---|---|---|---|
| Remote navigation | Audit and port behavior | Compose focus graph + virtual cursor | P0 |
| Gecko engine | Reuse lessons/delegates, refactor | Sole engine behind typed port | P0 |
| Tabs | Port models carefully | Lazy session restore and memory states | P0 |
| Omnibox/search | Rewrite parser/UI | Safe URI/search normalization | P0 |
| Back/forward/reload | Port behavior | Typed commands, tested Back policy | P0 |
| Bookmarks | Migrate schema or import | Room repository and folders | P0 |
| History | Port data rules | Privacy-aware Room repository | P0 |
| Downloads | Audit DownloadUtils | Gecko request -> scoped storage service | P0 |
| Fullscreen/media | Port delegate logic | TV media session, PiP, audio focus | P0 |
| Permissions/prompts | Rework | Central queued prompt coordinator | P0 |
| Content blocking | Reuse Gecko settings concepts | per-site controls and clear UI | P0 |
| Private browsing | Audit all writes | isolated profile, no normal persistence | P0 |
| Voice search | Port only after permission audit | platform voice input adapter | P1 |
| Shortcuts/home | Rebuild | native Compose home | P1 |
| User-agent switch | Port concept | global + per-site policy | P1 |
| Find in page | Implement via Gecko finder | remote-friendly result controls | P1 |
| Extensions | Expand existing built-in bridge | signed user-extension manager | P1 release blocker for stated requirement |
| Import/export | Rework | versioned JSON/HTML formats | P1 |
| Printing | Evaluate demand/device support | Post-v1 unless low-cost | P2 |
| Sync | Do not port | Post-v1 product decision | Out |

## 11. Data and storage design

### 11.1 Database tables

Proposed Room entities:

- `tabs`: normal-profile persistent tab metadata and snapshot reference;
- `bookmarks` and `bookmark_folders`;
- `history_entries` with normalized URL, title, visit time/count, transition type where available;
- `downloads` with URI-safe destination metadata and status;
- `site_permissions` keyed by normalized origin and permission type;
- `content_blocking_exceptions`;
- `extensions` product metadata and consent state;
- `shortcuts` for home layout;
- optional `schema_metadata` for imports/provenance.

Keep binary thumbnails/session snapshots in managed files with database references and atomic replacement, not giant Room blobs.

### 11.2 Settings

Use typed DataStore for:

- theme/color/motion/text scale;
- home/search engine;
- restore behavior;
- content-blocking level;
- download directory preference;
- UA/viewport default;
- developer settings in debug-only storage;
- retention periods and prompt behavior.

Never place secrets or arbitrary extension messages in plain preferences.

### 11.3 Import/export

- bookmarks: support interoperable HTML if feasible plus versioned mixi-bro JSON;
- settings: export only safe user-configurable values;
- history export is out by default due privacy;
- extension packages are not exported; export IDs/source references and require re-consent on reinstall;
- validate schema/version/size, use streaming parsers, and preview counts/conflicts before import;
- imports run transactionally with cancellation and rollback.

### 11.4 Clear-data semantics

Create a matrix mapping UI choices to local and Gecko storage:

- browsing history;
- cookies/site data;
- cache;
- permissions;
- downloads list versus downloaded files;
- form/session data;
- content-blocking statistics;
- extension data;
- all data.

The confirmation screen must state consequences. Tests verify both Gecko storage flags and local repositories are cleared consistently.

## 12. Privacy and security architecture

### 12.1 Default posture

- no analytics SDK;
- no telemetry unless later added through an ADR and explicit consent;
- strict or standard content blocking chosen during onboarding/settings with understandable tradeoffs;
- HTTPS-first behavior evaluated in the engine spike;
- release logging redacts URLs and titles;
- clipboard read only on explicit user action;
- external intents require allowlist/confirmation policy;
- FileProvider and scoped storage for files;
- exported Android components disabled unless required and permission-protected where possible;
- network security config prohibits cleartext except explicit local-network policy;
- WebView-related manifest metadata absent.

### 12.2 Permission model

Separate:

1. Android runtime permission;
2. site content permission;
3. extension permission;
4. app capability setting.

A camera request may require both Android camera permission and site permission. Denials must not loop. Persist site decisions with origin, scope, and expiry; provide a settings screen to revoke them.

### 12.3 Certificate and navigation safety

- Show origin/host separately from page title.
- Punycode/IDN display follows a reviewed anti-spoofing policy.
- Certificate errors default to no bypass; any advanced bypass is hidden behind a danger flow and never available for HSTS-equivalent cases.
- External schemes and app links are allowlisted; intent fallback URLs are parsed independently.
- Downloads never auto-open executables/packages.
- Local file access, content URIs, and `resource://` pages have explicit policy boundaries.

### 12.4 Supply chain

- Gradle dependency verification/checksums;
- pinned GitHub Actions SHAs or trusted major tags per policy;
- Dependabot/Renovate with grouped AndroidX updates and separate GeckoView PRs;
- secret scanning, CodeQL or equivalent static analysis, Android lint, dependency vulnerability scan, and license report;
- SBOM for release artifacts;
- protected signing environment and key rotation/recovery document;
- no arbitrary JitPack dependency without provenance review.

## 13. Detailed implementation phases

Phases are ordered by dependency, not calendar promises. Parallelize only tasks that do not write the same foundations.

### Phase 0 — repository and provenance bootstrap

**Goals**

- make the repository auditable and safe for multi-agent development;
- establish upstream provenance and fixed decisions.

**Tasks**

- initialize `main` and branch protection;
- add `AGENTS.md`, `.codex/config.toml`, and custom agent files;
- add this plan, ADR template, contribution guide, security policy, code of conduct if desired;
- import TV Bro source/history and tag the upstream baseline;
- add upstream license and notice;
- create initial issues/labels/milestones;
- configure `upstream` remote and document update workflow;
- decide package name, min SDK, distribution targets, and direct-GeckoView decision spike;
- add `.editorconfig`, Kotlin/Gradle formatting, dependency verification, and basic CI skeleton.

**Deliverables**

- bootstrapped repository with provenance;
- ADR-0001 through ADR-0003 accepted or marked pending spike;
- CI can validate Markdown/TOML/license presence.

**Exit criteria**

- no distributed binary yet;
- a new contributor or Codex session can understand scope and commands from repository files;
- TV Bro legal requirements are recorded.

**Lead agents**

- `architecture_explorer`, `test_release_engineer`, then `implementer` for isolated bootstrap files.

### Phase 1 — identity and build-system conversion

**Goals**

- produce a debug APK named mixi-bro that contains only GeckoView.

**Tasks**

- rename root project, modules if necessary, package declarations, namespace, app ID, labels, authorities, IDs, update metadata, and native extension channels;
- add original temporary development icon; final art comes later;
- remove WebView implementation, AndroidX WebKit dependency, WebView JS interface, engine selector, `geckoExcluded`, and all dead resources/tests;
- collapse engine flavor into stable/debug/nightly release policy;
- pin stable GeckoView 152 exact artifact after Maven metadata verification;
- migrate Java/Kotlin target to 17 and verify AGP/Kotlin compatibility;
- add Compose and TV Material dependencies without yet rewriting every screen;
- make About attribution available even in the transitional UI;
- configure ABI packaging and measure APK size.

**Deliverables**

- `assembleDebug` works;
- app installs alongside TV Bro;
- a simple Gecko page loads;
- no `android.webkit.WebView` usage in production sources;
- engine/version details visible in About.

**Exit criteria**

- source search and dependency graph prove GeckoView is the sole renderer;
- release build has no debug remote access/about:config;
- license/name/icon/app-ID requirements are met.

### Phase 2 — engine contract and one-tab vertical slice

**Goals**

- prove the architecture with one fully controlled tab before migrating every feature.

**Tasks**

- create `:core:model`, `:core:browser`, and `:engine:gecko` boundaries;
- implement runtime provider, session factory, view host, navigation/progress/security delegates;
- create a minimal Compose browser screen with omnibox and Back/Forward/Reload;
- add local HTTP/HTTPS test fixture server;
- handle load success, redirects, offline, DNS error, TLS error, external scheme, crash, and process recreation;
- implement developer engine diagnostics;
- benchmark runtime cold/warm initialization and first usable page.

**Deliverables**

- one-tab vertical slice using Compose + GeckoView;
- deterministic engine instrumentation suite;
- ADR confirming direct GeckoView or adopting Android Components based on measured spike.

**Exit criteria**

- no Activity or Composable owns long-lived session state;
- rotation/recreation does not duplicate runtime/session;
- critical engine events are typed and tested;
- Back behavior is documented.

### Phase 3 — tab state, persistence, and memory

**Goals**

- reliable multi-tab browsing under process death and constrained TV memory.

**Tasks**

- implement browser state machine/coordinator;
- add tab Room records and file-backed session snapshots;
- implement selected/warm/suspended/discarded lifecycle;
- create tab switcher with focus restoration and close/undo;
- lazy restore after process death;
- private-profile tab policy;
- memory-pressure handling and active-media exceptions;
- thumbnail capture budget and privacy filtering;
- race tests for rapid open/select/close/back operations.

**Deliverables**

- normal and private tabs;
- deterministic tab reducer tests;
- process-death and low-memory instrumentation tests;
- baseline memory report for representative devices.

**Exit criteria**

- tab order/selection survives restart;
- corrupt snapshots recover safely;
- private tabs leave no normal records;
- no session can be attached to two views.

### Phase 4 — Material 3 Expressive design system and adaptive shell

**Goals**

- replace transitional chrome with the final design foundation.

**Tasks**

- define color/type/shape/spacing/elevation/motion/focus tokens;
- implement TV and touch adaptive scaffolds;
- build reusable focused surface, icon button, action chip, dialog/panel, list/grid item, omnibox, tab card, status indicator;
- implement light/dark/high-contrast and reduced motion;
- create component catalog/previews and screenshot tests;
- define focus graphs and keyboard shortcuts;
- validate overscan, 720p/1080p/4K, font scaling, RTL, TalkBack;
- run usability passes for top-bar versus side-rail browser chrome.

**Deliverables**

- `:design-system` with stable wrappers;
- adaptive app shell and browser chrome;
- accessibility/focus acceptance checklist.

**Exit criteria**

- all browser actions reachable with D-pad and Back;
- focus is always visible and restored after overlays/tab switch;
- experimental Material APIs are isolated;
- representative screens pass screenshot/semantics tests.

### Phase 5 — core browser completeness

**Goals**

- complete day-to-day browsing before adding user extensions.

**Tasks**

- omnibox parser/search engines;
- find in page;
- desktop/mobile viewport and UA policy;
- zoom/text scaling;
- context/selection actions;
- prompt queue, HTTP auth, file picker/upload;
- Android and site permission coordinator;
- certificate/security information;
- content blocking level and per-site exceptions;
- fullscreen, media delegate, audio focus, TV media session, PiP;
- external intent/app-link policy;
- voice search adapter;
- local error and offline experiences.

**Deliverables**

- complete browsing vertical slice with settings;
- media and permission test suite;
- documented unsupported Gecko capabilities.

**Exit criteria**

- browser essentials matrix passes on API 26 and current target API;
- permission denial does not loop;
- fullscreen/PiP exits correctly with Back;
- no arbitrary `javascript:` URL execution path.

### Phase 6 — local data features

**Goals**

- ship home, bookmarks, history, downloads, and clear-data workflows.

**Tasks**

- Room schema and migration baseline;
- native Compose home/new-tab and shortcuts;
- bookmark folders/edit/search/import/export;
- history search/retention/private filtering;
- download pipeline using scoped storage, notifications, retry/open/share/delete;
- settings DataStore;
- site permissions manager;
- clear-data matrix and transactional implementation;
- About, notices, engine/build/license screens.

**Deliverables**

- all P0 local features;
- migration and import fuzz/size tests;
- required TV Bro attribution in final UI.

**Exit criteria**

- clear-data behavior matches wording;
- downloaded file deletion is never confused with removing list history;
- import failures are atomic;
- no private data appears in home/history/thumbnails.

### Phase 7 — built-in extension bridge hardening

**Goals**

- replace legacy bridge assumptions with a minimal, audited protocol.

**Tasks**

- inventory every existing TV Bro built-in extension capability;
- remove capabilities now covered by native GeckoView APIs;
- rename extension ID and native channel names;
- define versioned message schemas and payload limits;
- use `ensureBuiltIn` and reconcile upgrade failures;
- isolate debug test channels from release;
- add content-script origin checks and per-session routing;
- negative/fuzz tests for messages;
- document why each permission exists.

**Deliverables**

- minimal built-in bridge and protocol specification;
- security review report;
- graceful behavior when bridge install/connect fails.

**Exit criteria**

- no Android-supplied arbitrary JS execution;
- no wildcard native message handler;
- all messages validated and mapped to a tab/session;
- security reviewer signs off.

### Phase 8 — user WebExtensions MVP

**Goals**

- satisfy extension support with a safe, understandable manager.

**Tasks**

- implement extension repository/controller reconciliation;
- approved source and signed-XPI validation;
- install prompt with API/host/data-collection permissions;
- installed list/details;
- enable/disable/uninstall;
- update and increased-permission handling;
- private browsing toggle;
- action metadata, pinning, badge, popup host, options page;
- compatibility/error state mapping;
- curated fixture extensions and AMO test set;
- crash-loop and extension-process recovery;
- extension storage/clear-data behavior.

**Deliverables**

- signed extension end-to-end flow;
- compatibility matrix;
- threat model and adversarial test report;
- user documentation explaining limitations.

**Exit criteria**

- permissions cannot be skipped;
- unsigned production install fails safely;
- private access defaults off and is enforced;
- disable/uninstall/update state survives restart;
- action popup is fully D-pad operable;
- no critical/high security findings remain.

### Phase 9 — quality, performance, and hardening

**Goals**

- turn feature-complete software into a releasable browser.

**Tasks**

- Baseline Profiles and startup optimization after measurement;
- Macrobenchmark startup/tab switch/scroll/chrome motion;
- memory tests across tab states and extension load;
- battery/network idle behavior;
- APK/AAB size analysis and ABI strategy;
- crash recovery, ANR, StrictMode, leaked Activity/View checks;
- accessibility audit and real-TV remote testing;
- localization pseudo-locale and RTL passes;
- security review of intents, providers, permissions, network config, storage, logs, extensions;
- dependency/license/SBOM scan;
- release logging and diagnostics review;
- backup/restore behavior review.

**Provisional budgets**

These are starting gates to calibrate on representative hardware, not marketing claims:

- no main-thread network or database access;
- no repeated runtime creation;
- tab switch to already warm session should feel immediate and remain within the frame/jank budget;
- cold startup and first-page usability tracked at P50/P95 on a low-end target TV;
- memory growth per additional warm tab measured and capped by suspension policy;
- release artifact size tracked with a fail-on-regression threshold after the first stable baseline;
- zero known critical/high security defects and zero reproducible data-loss defects.

**Exit criteria**

- release candidate passes the full device/feature matrix;
- performance regressions are visible in CI or release reports;
- all release-blocking findings resolved.

### Phase 10 — release and maintenance

**Goals**

- reproducible signed delivery and sustainable engine updates.

**Tasks**

- protected release workflow and signing setup;
- versioning/changelog/release notes;
- GitHub release artifacts and checksums;
- store metadata, privacy policy, Data safety declarations, TV screenshots/banner;
- staged rollout and rollback plan;
- crash/support intake without collecting browsing data;
- GeckoView stable update playbook;
- monthly/each-train extension compatibility sweep;
- security disclosure process and supported-version policy;
- upstream TV Bro comparison cadence for useful remote-control fixes.

**Exit criteria**

- signed artifact provenance is documented;
- rollback can restore the previous known-good release;
- one simulated GeckoView update PR completes the full compatibility gate;
- maintenance ownership is assigned.

## 14. Testing strategy

### 14.1 Test pyramid

#### Pure unit tests

- omnibox/URI parser;
- browser/tab reducer;
- permission and prompt state machines;
- extension permission humanization and policy;
- download state transitions;
- clear-data mapping;
- import/export validation;
- redaction/logging;
- focus destination calculations where modeled.

#### JVM/Robolectric tests

- repositories and ViewModels where Android behavior is required;
- Room DAOs/migrations;
- DataStore serializers;
- intent and FileProvider policy helpers;
- Compose semantics where reliable.

#### Instrumentation tests

- GeckoRuntime/session lifecycle;
- navigation/redirect/error/TLS fixtures;
- view attach/detach and tab switching;
- process/activity recreation;
- permissions/prompts/file picker;
- media/fullscreen/PiP;
- WebExtension install/actions/private mode;
- scoped storage downloads/uploads;
- D-pad and keyboard behavior.

#### Macrobenchmark/manual hardware

- cold/warm startup;
- first usable page;
- tab switching and memory pressure;
- long browsing/media sessions;
- low-end Android TV devices;
- remote variants, keyboards, mice, gamepads;
- DRM services using legal test content;
- accessibility/TalkBack.

### 14.2 Deterministic web fixture server

Create a local fixture app/server containing:

- normal HTML, slow response, redirects, large page, history navigation;
- HTTP auth;
- permissions: camera/microphone/geolocation/notifications/clipboard where supported;
- file upload/download;
- fullscreen/video/audio;
- service worker/storage/cookies;
- mixed content and test certificates;
- external scheme links;
- popup/new-window requests;
- crash/recovery test hooks where feasible;
- extension fixture pages.

Gating CI must not rely on public websites. A non-gating compatibility job may probe a small documented site set.

### 14.3 API/device matrix

Minimum automated matrix:

- API 26 emulator;
- one mid API representative, for example API 30/31;
- API 36 current target;
- Android TV/Google TV emulator profile;
- phone profile for adaptive regressions;
- arm64 physical TV in release candidate testing;
- x86_64 emulator artifacts.

Evaluate armeabi-v7a only after confirming GeckoView availability, device demand, store constraints, size, and performance. Do not inherit ABI support blindly.

### 14.4 Extension fixtures

Maintain controlled signed or built-in test fixtures for:

- no-permission extension;
- host permission;
- optional permission;
- browser action and popup;
- options page;
- content script;
- update with same permissions;
- update with increased permissions;
- incompatible API;
- private browsing;
- native-message rejection;
- crash/invalid package cases.

Do not commit private signing material. Use public test fixtures or CI-generated debug-only built-ins according to Mozilla signing constraints.

## 15. CI/CD design

### 15.1 Pull-request checks

- Gradle wrapper validation;
- formatting/Spotless or ktlint;
- Android lint;
- unit and JVM tests;
- Room schema/migration verification;
- debug assemble for supported ABIs;
- dependency verification and license report;
- secret scan;
- static security analysis;
- affected instrumentation smoke job where infrastructure permits;
- Markdown/TOML/JSON validation;
- source check forbidding `android.webkit.WebView` in production.

### 15.2 Scheduled checks

- dependency update discovery;
- stable GeckoView update branch/PR;
- nightly/beta compatibility build, non-blocking for release branch;
- full instrumentation matrix;
- extension curated-set compatibility;
- vulnerability database scan;
- benchmark trend report.

### 15.3 Release checks

- clean checkout, locked dependencies, reproducible unsigned build;
- full tests and lint;
- release minification mapping retained securely;
- SBOM and notices;
- artifact signing in protected environment;
- checksum and provenance statement;
- install/upgrade/rollback smoke tests;
- About page version/engine/license verification;
- no debug menus, remote debugging, `about:config`, test certificates, or developer endpoints.

### 15.4 Branch and review policy

- protected `main`;
- normal work on `agent/<description>` or feature branches;
- draft PR by default for agent-generated work;
- at least one human approval for release/security-sensitive changes;
- engine updates require `geckoview_specialist` review;
- extension changes require `extension_security_reviewer` review;
- UI/focus changes require `material3_tv_designer` review;
- migrations/release workflows require `test_release_engineer` review;
- do not merge failing or skipped required checks.

## 16. Multi-agent execution plan

The repository includes six custom Codex roles and a maximum of six concurrent threads with one level of delegation.

### 16.1 Standard feature workflow

Lead thread:

1. Read `AGENTS.md`, this plan, and relevant ADRs.
2. State acceptance criteria and file/module boundaries.
3. Spawn in parallel:
   - `architecture_explorer` to map current code and provenance;
   - one domain specialist (`geckoview_specialist`, `material3_tv_designer`, or `extension_security_reviewer`);
   - `test_release_engineer` to design validation if the change affects CI, persistence, packaging, or broad behavior.
4. Wait and synthesize one design decision.
5. Assign one bounded writer (`implementer`) per non-overlapping module/file set.
6. Run tests and inspect the combined diff.
7. Ask the domain specialist to review the diff read-only.
8. Fix findings serially and prepare the PR summary.

### 16.2 Example: implement tab suspension

Parallel investigation:

- `architecture_explorer`: map current tab/session persistence and ownership;
- `geckoview_specialist`: verify state flush/restore/close and form/media constraints;
- `test_release_engineer`: design process-death and memory-pressure tests.

Synthesis:

- lead defines active/warm/suspended/discarded states and ownership invariant.

Writing:

- one implementer owns `:core:browser` reducer/models;
- a second implementer may own `:engine:gecko` only in an isolated worktree after interfaces are frozen;
- test writer owns instrumentation fixture files, not production files.

Review:

- Gecko specialist audits lifecycle races and API assumptions.

### 16.3 Example: extension install screen

Parallel investigation:

- `extension_security_reviewer`: threat model and permission rules;
- `material3_tv_designer`: focus graph and comprehension layout;
- `geckoview_specialist`: install prompt fields/error codes/threading;
- `architecture_explorer`: current extension bridge and data flow.

Writing is serialized across shared extension policy and UI state. Do not let separate agents independently invent permission models.

### 16.4 Conflict control

- never assign two writers the same file;
- version catalog, settings Gradle, shared design tokens, Room schema, and release workflow have one writer at a time;
- use worktrees for parallel implementation only after interfaces are committed;
- read-only agents may overlap freely;
- the lead, not a subagent, resolves conflicting recommendations;
- subagents may not spawn deeper subagents because `max_depth = 1`.

## 17. Initial issue/epic backlog

Create these as epics or milestones, then split into acceptance-testable issues.

### E00 — repository governance and provenance

- import/tag upstream source;
- license/notice/About requirements;
- AGENTS and subagent config;
- ADR template and contribution/security docs;
- branch protections and issue labels.

### E01 — mixi-bro identity

- package/application ID rename;
- product strings and authorities;
- icon/brand design;
- update/signing identity separation;
- About attribution.

### E02 — Gecko-only build

- remove WebView and engine flavor;
- pin stable GeckoView;
- build types/channels;
- Java 17 and toolchain validation;
- ABI and artifact-size baseline.

### E03 — browser engine core

- runtime provider;
- session factory;
- delegate adapters;
- Compose GeckoView host;
- diagnostics and fixture server.

### E04 — tabs and restoration

- browser reducer;
- tab persistence;
- session snapshots;
- tab switcher;
- private tabs;
- memory policy.

### E05 — design system and app shell

- tokens and component catalog;
- TV/touch adaptive shell;
- focus graphs and shortcuts;
- accessibility/reduced motion;
- visual test infrastructure.

### E06 — browser essentials

- omnibox;
- prompts/permissions/security;
- content blocking;
- find/zoom/UA;
- media/fullscreen/PiP;
- voice/external intents/error pages.

### E07 — local data features

- Room/DataStore;
- home/shortcuts;
- bookmarks/history;
- downloads;
- site data/clear data;
- import/export.

### E08 — built-in extension bridge

- capability inventory;
- renamed/versioned extension;
- typed messaging;
- negative tests and security review.

### E09 — user extensions

- install source/prompt;
- manager/details;
- lifecycle/update/private access;
- actions/popups/options;
- compatibility and security suite.

### E10 — release readiness

- performance/memory/size;
- CI matrix;
- security/accessibility/localization;
- SBOM/notices/signing;
- store/GitHub release and maintenance playbook.

## 18. Decision register

Decide these explicitly; recommended defaults are shown.

| Decision | Recommended default | Deadline |
|---|---|---|
| Source strategy | Preserve/import TV Bro history, then modernize | Phase 0 |
| Package ID | `io.github.codexclamps.mixibro` | Phase 0 |
| Engine abstraction | Direct GeckoView behind product-level port | Phase 2 spike |
| Production channel | Latest tested stable GeckoView only | Phase 1 |
| minSdk | 26 | Phase 1 |
| UI | Compose + M3 + TV Material | Phase 1 |
| Native home page | Compose, not injected web page | Phase 4/6 |
| Private tab restore | Do not restore after process death | Phase 3 |
| Extension source | Mozilla-signed, approved HTTPS/AMO path | Phase 8 |
| Unsigned extensions | Debug developer mode only | Phase 8 |
| Telemetry | None by default | Phase 0 |
| Phone/tablet | Adaptive support in same app, TV remains primary | Phase 4 |
| 32-bit ABI | Evidence-based; not guaranteed | Phase 1/9 |
| Distribution | GitHub first; store/F-Droid after policy review | Phase 10 |
| Sync | Out of v1 | Post-v1 |

## 19. Risk register

### R1 — GeckoView artifact size and TV storage

**Impact:** large APK/AAB and slow install/update.  
**Mitigation:** ABI splits/app bundles, size tracking, remove dual engine and dead resources, evaluate supported ABIs, avoid unnecessary Android Components.

### R2 — memory pressure with many tabs

**Impact:** process kills, crashes, media interruption.  
**Mitigation:** active/warm/suspended/discarded states, lazy restore, benchmark low-end devices, protect active media/forms, explicit memory policy.

### R3 — Compose/GeckoView focus conflict

**Impact:** remote traps, lost focus, unusable content.  
**Mitigation:** dedicated host, explicit focus boundaries, virtual cursor fallback, focus tests, real hardware passes, keep session state outside composition.

### R4 — extension API incompatibility

**Impact:** users expect desktop add-ons that fail.  
**Mitigation:** curated/tested list, compatibility state, clear limitations, fixtures, no “all Firefox extensions” marketing claim.

### R5 — extension security/native messaging

**Impact:** page data or native capability compromise.  
**Mitigation:** signed source, permission prompts, allowlisted extension/channel, typed messages, private mode opt-in, dedicated security reviewer and negative tests.

### R6 — unstable Material Expressive API adoption

**Impact:** frequent source breakage.  
**Mitigation:** stable M3/TV dependencies, wrappers around experimental APIs, token-driven custom equivalents, screenshot/focus tests.

### R7 — dual-source migration complexity

**Impact:** long-lived legacy code and duplicate behavior.  
**Mitigation:** remove WebView early, vertical slice before broad migration, delete dead code per feature, prohibit new legacy UI.

### R8 — TV Bro license violation

**Impact:** distribution cannot continue.  
**Mitigation:** preserved license/history, new identity, required About credit, notices, release checklist.

### R9 — engine update regressions

**Impact:** security versus compatibility tension.  
**Mitigation:** scheduled stable update PR, deterministic web/extension suite, beta/nightly preview build, rollback-ready releases.

### R10 — DRM/service compatibility

**Impact:** expected streaming sites fail.  
**Mitigation:** legal test matrix, report capabilities accurately, avoid unsupported promises, validate Widevine/media behavior by engine/device.

### R11 — store policy conflicts

**Impact:** rejection due self-update, extension behavior, downloads, privacy declarations.  
**Mitigation:** separate distribution policy, no built-in self-update in store build unless allowed, policy review before submission, transparent Data safety form.

### R12 — over-modularization slows delivery

**Impact:** Gradle overhead and abstraction churn.  
**Mitigation:** create modules at proven ownership boundaries, begin with core/engine/design/app, extract feature modules when active.

## 20. Version-one release acceptance criteria

A v1.0 release candidate is acceptable only when all of the following are true:

### Identity and legal

- launcher and app identify only as mixi-bro;
- unique icon/package/application ID/signing identity;
- TV Bro license retained;
- About contains required credit and source link;
- third-party notices accessible.

### Engine

- stable GeckoView is the only renderer;
- one runtime per process and one session per live tab invariant tested;
- no release remote debugging/about:config;
- engine version visible;
- process/crash recovery works.

### Browsing

- navigation, tabs, restore, private tabs, bookmarks, history, downloads, file upload, prompts, permissions, security state, content blocking, find, zoom, UA, fullscreen/media/PiP, voice where available, and external intents meet acceptance tests;
- Back and D-pad behavior is deterministic;
- no critical data-loss bug.

### Extensions

- signed install and permission review;
- enable/disable/uninstall/update;
- private access off by default;
- action popup/options usable with remote;
- unsupported APIs clearly surfaced;
- built-in native messaging allowlisted and typed;
- no unresolved critical/high security issue.

### UI/accessibility

- coherent M3 Expressive identity;
- stable focus and restoration on TV;
- touch/keyboard/mouse paths work where supported;
- TalkBack labels, contrast, font scale, RTL, reduced motion, overscan validated;
- no blocking jank regression on target devices.

### Quality/release

- required CI green;
- deterministic engine/extension fixtures green;
- API/device/ABI release matrix complete;
- performance/memory/size baselines recorded;
- release artifact signed in protected CI with SBOM/notices/checksum;
- privacy/store metadata match actual behavior;
- rollback and GeckoView update playbooks tested.

## 21. First actionable development slice

After repository access and source import are ready, the first PR-sized implementation slice should be narrowly defined:

**Title:** `Bootstrap mixi-bro identity and Gecko-only build`

**Scope:**

- import TV Bro 2.1.6 history or source baseline;
- apply new root project/app/package identity;
- retain license and add notice/About attribution;
- remove `geckoExcluded` and make `:app:gecko` unconditional;
- remove Android WebView implementation/dependency from the active build;
- pin the latest stable GeckoView artifact available on the implementation date;
- add Java 17 compatibility and basic CI assemble/test/lint;
- do not yet rewrite every screen in Compose.

**Acceptance:**

- debug APK installs as `io.github.codexclamps.mixibro` alongside TV Bro;
- launcher shows mixi-bro with non-TV-Bro temporary art;
- a page loads through GeckoView;
- About displays TV Bro attribution and Gecko build version;
- source/dependency search finds no production WebView renderer;
- debug build passes lint/unit/assemble checks;
- release configuration disables engine debugging.

This slice creates a trustworthy base. The next PR should be the one-tab Compose/engine-contract vertical slice, not a large visual rewrite mixed with package migration.

## 22. Source references

Current decisions were based on the following primary sources and should be rechecked when versions change:

- TV Bro repository: <https://github.com/truefedex/tv-bro>
- GeckoView overview: <https://mozilla.github.io/geckoview/>
- GeckoView quick start: <https://mozilla.github.io/geckoview/consumer/docs/geckoview-quick-start>
- GeckoView API: <https://mozilla.github.io/geckoview/javadoc/mozilla-central/>
- Firefox Android architecture: <https://firefox-source-docs.mozilla.org/mobile/android/>
- Firefox release notes: <https://developer.mozilla.org/en-US/docs/Mozilla/Firefox/Releases>
- Android Material 3 Compose: <https://developer.android.com/develop/ui/compose/designsystems/material3>
- Compose release versions: <https://developer.android.com/jetpack/androidx/releases/compose>
- Compose for TV releases: <https://developer.android.com/jetpack/androidx/releases/tv>
- Compose focus behavior: <https://developer.android.com/develop/ui/compose/touch-input/focus/change-focus-behavior>
- Codex `AGENTS.md`: <https://learn.chatgpt.com/docs/agent-configuration/agents-md>
- Codex subagents: <https://learn.chatgpt.com/docs/agent-configuration/subagents>

