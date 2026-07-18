# mixi-bro

mixi-bro is a planned remote-first Android browser built on Mozilla GeckoView with a Material Design 3 Expressive interface and safe WebExtension support.

> Repository status: architecture and agent-workflow bootstrap. Application source should be imported from the pinned TV Bro baseline and modernized according to the implementation plan; this bootstrap does not yet contain a buildable Android app.

## Product direction

- Android TV / Google TV first, with adaptive keyboard, mouse, and touch support.
- GeckoView is the only production rendering engine.
- Stable GeckoView is pinned and updated through compatibility-tested pull requests.
- Jetpack Compose, Material 3, and stable TV Material provide the UI foundation.
- Signed user-installed WebExtensions receive explicit permission and private-mode controls.
- Privacy, remote focus, session restoration, and memory pressure are release-blocking concerns.

## Read first

- [`AGENTS.md`](AGENTS.md) - repository-wide engineering, licensing, security, and multi-agent rules.
- [`docs/IMPLEMENTATION_PLAN.md`](docs/IMPLEMENTATION_PLAN.md) - detailed architecture, migration, milestones, test matrix, risks, and v1 acceptance criteria.
- [`docs/THREE_PERSON_WORK_PLAN.md`](docs/THREE_PERSON_WORK_PLAN.md) - three-worker ownership, dependency, handoff, and integration plan.
- [`.codex/config.toml`](.codex/config.toml) - project subagent concurrency and nesting limits.
- [`.codex/agents/`](.codex/agents/) - read-only specialist roles plus one generic and three path-scoped writable implementation roles.
- [`docs/adr/`](docs/adr/) - initial architecture decisions.
- [`NOTICE.md`](NOTICE.md) and [`licenses/TV_BRO_LICENSE.md`](licenses/TV_BRO_LICENSE.md) - upstream provenance and the retained TV Bro license.

## Upstream attribution

mixi-bro is intended to derive from [TV Bro](https://github.com/truefedex/tv-bro), originally by Fedir Tsapana. Before source or binaries are distributed, retain the upstream license, use a different app name/icon/application ID, and include the license-required in-app About attribution. See the legal and provenance section of the implementation plan.

GeckoView is a Mozilla technology. mixi-bro is not Firefox and must not use Mozilla or Firefox trademarks or imply endorsement.

## Initial implementation order

1. Import and tag the TV Bro 2.1.6 baseline with provenance intact.
2. Apply mixi-bro identity and required attribution.
3. Remove the WebView engine and make GeckoView unconditional.
4. Establish a typed browser core and one-tab Compose/GeckoView vertical slice.
5. Add reliable tabs, persistence, private mode, and memory suspension.
6. Build the M3 Expressive adaptive TV shell and complete browser essentials.
7. Add local data features, then harden the built-in extension bridge.
8. Add signed user WebExtensions and complete security, compatibility, performance, and release gates.

## Using the project agents

The root thread is the lead. It investigates and synthesizes first, freezes one base SHA, then assigns up to three writable agents to separate branches and worktrees created from that base. The workers can edit and commit concurrently because their path allowlists do not overlap.

| Worker role | Exclusive write ownership |
| --- | --- |
| `platform_engine_implementer` | `core/common/**`, `core/model/**`, `core/browser/**`, `engine/gecko/**` |
| `product_ui_implementer` | `design-system/**`, `feature/**` |
| `data_extension_release_implementer` | `core/data/**`, `core/extensions/**`, `testing/web-fixtures/**`, `benchmark/**`, `.github/workflows/**`, `docs/security/**`, `docs/release/**` |

Each worker receives an absolute worktree path, assigned branch, immutable base SHA, and path allowlist. It stages only explicit owned files, creates one focused commit on its worker branch, and reports the commit SHA and observed checks. Workers never merge, rebase, pull, push, or use `git add .`; the lead reviews and integrates their commits. Any cross-owner file is reassigned or serialized by the lead rather than edited concurrently.

A useful Codex prompt for a non-trivial change is:

```text
Read AGENTS.md, docs/IMPLEMENTATION_PLAN.md, and docs/THREE_PERSON_WORK_PLAN.md. Use relevant read-only specialists for investigation, wait for their reports, and synthesize one implementation decision. Freeze one base SHA, create the platform, product, and data-trust worktrees, then assign only non-overlapping allowlisted edits to platform_engine_implementer, product_ui_implementer, and data_extension_release_implementer. Wait for all three commit SHAs, review their changed paths and observed checks, then integrate from the lead branch.
```

Codex loads the project roles from `.codex/agents/*.toml`. Configuration permits one lead plus three direct agents at a time; recursive fan-out is disabled. Run read-only specialist investigations before occupying all three worker slots for implementation.

## Proposed package and platform baseline

- application ID: `io.github.codexclamps.mixibro`
- minimum Android API: 26
- Java: 17
- language/build: Kotlin and Gradle Kotlin DSL
- production engine: latest tested stable GeckoView, pinned exactly
- UI: Jetpack Compose, Material 3, TV Material

These are controlled by ADRs and may change only through an explicit, documented decision.