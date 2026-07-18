# ADR-0004: Three-agent parallel ownership and octopus integration

- **Status:** Accepted
- **Date:** 2026-07-18
- **Decision owners:** mixi-bro maintainers

## Context

Engine, product UI, persistence, extension trust, fixtures, and release work can proceed in parallel, but unrestricted writers would collide on contracts, build files, app wiring, and policy models. Merge-time conflict resolution would make the integrator an unreviewed fourth implementer.

## Decision

### Exclusive lanes

| Lane | Exclusive writable paths |
|---|---|
| Platform / `platform_writer` | `core/common/**`, `core/model/**`, `core/browser/**`, `engine/gecko/**` |
| Product / `product_writer` | `design-system/**`, `feature/**`, including `feature/extensions/**` |
| Data/trust / `data_trust_writer` | `core/data/**`, `core/extensions/**`, `testing/web-fixtures/**`, `benchmark/**`, `.github/workflows/**`, `docs/security/**`, `docs/release/**` |

The lead/integration captain exclusively owns `app/**`; root Gradle/settings/version-catalog/wrapper/verification/build-logic files; `.codex/**`; repository governance files; shared roadmaps and ADRs; and licensing, notice, attribution, and third-party-license files. A lane requests a handoff rather than editing another owner's path.

### Module boundaries

`:core:extensions` owns engine-neutral extension identity, source trust, requested/granted permissions, compatibility, install/update, enablement, and private-access policy and ports.

`:testing:web-fixtures` owns deterministic local HTTP/HTTPS, TLS, media, upload/download, permission, popup, error, and extension fixtures and is test-only.

`:engine:gecko` implements the extension ports with GeckoView controller and bridge APIs. `:feature:extensions` owns presentation. `:core:browser` retains browser lifecycle and site-permission contracts. GeckoView types do not cross into `:core:extensions` or `:feature:extensions`.

### Worktrees and integration

The lead freezes shared contracts and integration files in a base commit, then creates `agent/<slice>-platform`, `agent/<slice>-product`, and `agent/<slice>-data-trust` worktrees from that exact revision.

Each writer commits only allowlisted paths. Before merging, the lead verifies common ancestry, allowlist compliance, pairwise-disjoint changed-path sets, no lead-only changes, focused checks, and completed handoffs.

All three completed heads are merged in one octopus merge commit. If a conflict occurs, the lead aborts the merge. Lane-local problems return to the owner branch; shared-contract problems return to a revised common base. Conflicts are never resolved ad hoc in the integration worktree.

## Consequences

Benefits include simultaneous commits without shared files, visible ownership, clean engine/UI/trust boundaries, and one integration commit recording all delivered heads.

Costs include an up-front contract freeze, possible pauses when contracts are incomplete, lead-owned build/app bottlenecks, and rejection of path overlap even when Git could merge it.

## Validation

Record the base and three head commits, per-lane path lists, allowlist and disjointness checks, focused and combined test results, required specialist reviews, residual risks, and unrun checks.

## Rollback

Before integration, recreate only the defective lane from the recorded base. After integration, revert the octopus merge as one unit when the combined delivery must be removed. Replace this topology only through a superseding ADR and coordinated updates to `AGENTS.md` and the implementation plan.
