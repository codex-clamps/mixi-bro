# ADR-0001: GeckoView is the sole production web engine

- Status: Proposed; accept after the Phase 2 direct-GeckoView versus Android Components spike
- Date: 2026-07-18
- Owners: mixi-bro maintainers

## Context

TV Bro historically supports Android WebView and has a Gecko-included flavor. mixi-bro's defining requirement is a current Firefox/Gecko engine with WebExtension support. Maintaining WebView and Gecko doubles lifecycle, permission, download, media, security, and test behavior and makes extension support inconsistent.

## Decision

Normal mixi-bro tabs use GeckoView only. The app owns one process-wide `GeckoRuntime`, creates one `GeckoSession` per live tab, and attaches the selected session to a `GeckoView` hosted from Compose. Product code depends on engine-neutral browser ports; Gecko-specific types stay in `:engine:gecko`.

The first vertical-slice spike compares direct GeckoView with Mozilla Android Components. The default is direct GeckoView because it preserves control and fits the custom TV UI. Android Components may replace parts of the adapter only if measured evidence shows lower lifecycle/extension risk without importing inappropriate frontend assumptions. Android WebView is not a fallback option.

## Consequences

### Positive

- one rendering/security model;
- predictable bundled engine version;
- direct WebExtensionController support;
- smaller test matrix than a dual engine;
- clearer product identity.

### Negative / tradeoffs

- larger application artifact than relying on system WebView;
- every security update requires an app dependency update and release;
- GeckoView lifecycle, delegates, memory, and compatibility are the app's responsibility;
- some sites or DRM scenarios may differ from Chromium-based TV browsers.

## Alternatives considered

- Keep WebView as a fallback: rejected because it creates two products and breaks extension expectations.
- Fork Firefox for Android/Fenix: rejected initially because it carries a much larger frontend/product surface than required.
- Use Android Components wholesale: pending spike; may be adopted selectively, not as a second engine.

## Validation and rollback

Validate with the deterministic engine suite, API/device matrix, extension fixtures, and size/memory benchmarks. A GeckoView train may be rolled back to the previous tested stable artifact for a release-blocking regression, with the security impact documented. The architecture does not roll back to WebView.
