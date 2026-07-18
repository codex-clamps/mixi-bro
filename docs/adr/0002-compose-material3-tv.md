# ADR-0002: Compose Material 3 Expressive with TV-first adaptive components

- Status: Proposed
- Date: 2026-07-18
- Owners: mixi-bro maintainers

## Context

TV Bro's existing UI is ViewBinding/XML based. mixi-bro needs an original Material Design 3 Expressive identity and dependable remote focus while remaining usable with keyboard, mouse, and touch.

## Decision

New UI is written in Jetpack Compose. Stable Material 3 and stable TV Material APIs are preferred. A `:design-system` module owns tokens and wrappers. Experimental expressive APIs are isolated behind local wrappers and opt-ins. GeckoView remains an Android View hosted with `AndroidView`; its session and browser state are not owned by composition.

The default layout is TV/landscape and D-pad first. Adaptive scaffolds may present touch-appropriate chrome on phones/tablets while sharing domain state and feature logic.

## Consequences

### Positive

- consistent design tokens and reusable focus behavior;
- easier adaptive layouts and semantics testing;
- clear path away from legacy XML screens;
- modern motion, typography, and shape system.

### Negative / tradeoffs

- GeckoView interop and focus transfer require careful lifecycle handling;
- TV Material and phone Material components cannot be mixed casually;
- experimental expressive APIs can change;
- migration must be staged to avoid a long mixed-UI period.

## Alternatives considered

- Restyle XML/AppCompat: rejected because it does not meet the desired design-system/adaptive direction.
- Compose phone UI scaled for TV: rejected because ten-foot focus and density need dedicated components.
- Fully custom canvas UI: rejected because accessibility and platform integration costs are unnecessary.

## Validation and rollback

Validate every screen with focus graphs, Compose semantics, TalkBack, RTL, font scaling, reduced motion, overscan, 720p/1080p/4K, and physical remote tests. Experimental component wrappers can fall back to stable/custom equivalents without changing feature APIs.
