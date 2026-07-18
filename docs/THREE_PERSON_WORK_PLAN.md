# Three-Agent Parallel Delivery Plan

This plan defines three concurrent write agents plus one lead/integration captain. The lead is the coordination and integration lane, not a fourth feature lane. The three agents may commit at the same time only from separate Git worktrees and only within their exclusive path ownership.

## Current status

- The repository is planning-only.
- TV Bro source has not been imported.
- No buildable Android application or Gradle module tree exists yet.
- The modules, branches, worktrees, and commands below are planned state, not existing state.
- Serialized bootstrap and the contract-freeze gate must finish before parallel implementation begins.

## Source documents

- [Repository overview](../README.md)
- [Bootstrap and import procedure](../APPLY_TO_REPOSITORY.md)
- [Implementation roadmap](IMPLEMENTATION_PLAN.md)
- [ADR-0001: GeckoView-only engine](adr/0001-geckoview-only-engine.md)
- [ADR-0002: Compose and Material 3 TV](adr/0002-compose-material3-tv.md)
- [ADR-0003: WebExtension trust model](adr/0003-webextension-trust-model.md)
- [ADR-0004: Three-agent parallel ownership](adr/0004-three-agent-parallel-ownership.md)

## Operating model

| Role | Delivery responsibility | Integration responsibility |
|---|---|---|
| Lead / Integration Captain | Serialized bootstrap, frozen contracts, application wiring, shared build configuration, and final acceptance | Creates the common base and worktrees, verifies paths, runs gates, and creates the octopus integration merge |
| Agent 1: Platform / Engine | Engine-neutral browser foundations and GeckoView implementation | Delivers one platform branch and reports its commit SHA and changed paths |
| Agent 2: Product UI | Design system, adaptive feature UI, D-pad behavior, accessibility, and localization | Delivers one product branch and reports its commit SHA and changed paths |
| Agent 3: Data / Extension Trust / Release | Persistence, extension trust domain, shared web fixtures, CI, legal, and release controls | Delivers one data-trust branch and reports its commit SHA and changed paths |

- The lead does not edit an agent-owned path during an active wave.
- An agent does not edit the lead path or another agent's path.
- Any path not explicitly assigned below is lead-only until the next contract freeze assigns it.
- Required read-only specialist reviews are separate from the three write lanes and never modify worker branches.

## Planned physical project structure

~~~text
app/                         :app
design-system/               :design-system
core/common/                 :core:common
core/model/                  :core:model
core/browser/                :core:browser
core/extensions/             :core:extensions
core/data/                   :core:data
engine/gecko/                :engine:gecko
feature/browser/             :feature:browser
feature/home/                :feature:home
feature/tabs/                :feature:tabs
feature/bookmarks/           :feature:bookmarks
feature/history/             :feature:history
feature/downloads/           :feature:downloads
feature/extensions/          :feature:extensions
feature/settings/            :feature:settings
feature/about/               :feature:about
testing/web-fixtures/        :testing:web-fixtures
benchmark/                   :benchmark
~~~

:core:extensions is the engine-neutral extension trust and lifecycle domain. It owns signed-source policy, permission models, compatibility state, private-access consent, update state, and extension ports. It contains no GeckoView types and no Compose UI.

:testing:web-fixtures is a test-only module for deterministic HTTP/HTTPS fixtures, certificates, redirects, downloads, uploads, authentication, media metadata, and malicious extension-package cases. Production release configurations must not depend on it.

## Planned dependency map

| Module | May depend on | Must not depend on |
|---|---|---|
| :core:common | Kotlin and approved small utilities | Android UI, GeckoView, feature modules |
| :core:model | :core:common | GeckoView, Compose, data implementations |
| :core:browser | :core:common, :core:model | GeckoView, Compose, feature modules |
| :core:extensions | :core:common, :core:model, :core:browser | GeckoView, Compose, extension UI |
| :core:data | :core:common, :core:model, :core:browser, :core:extensions | GeckoView and feature modules |
| :engine:gecko | :core:common, :core:model, :core:browser, :core:extensions | Feature UI and concrete data implementation details |
| :design-system | :core:common and :core:model when required | GeckoView, data implementations, feature modules |
| :feature:* | :design-system and required core contract modules | :engine:gecko, concrete Room/DataStore implementations, feature-to-feature dependencies unless approved |
| :testing:web-fixtures | Test libraries and minimal core models | Production feature or application wiring |
| :app | Feature modules, :engine:gecko, and :core:data for dependency wiring | Business logic that belongs in core or feature modules |
| :benchmark | :app and benchmark tooling | Shipping application logic |

The allowed direction is feature/UI -> core contracts -> data or engine implementation, with :app as the composition root. A dependency exception requires an ADR update before the next base is frozen.

## Exact exclusive path ownership

Ownership applies after the serialized bootstrap base is committed.

| Owner | Exclusive writable paths |
|---|---|
| Lead / Integration Captain | app/**; settings.gradle.kts; root build.gradle.kts; gradle.properties; gradle/libs.versions.toml; gradle/wrapper/**; build-logic/**; .gitignore; AGENTS.md; README.md; APPLY_TO_REPOSITORY.md; docs/IMPLEMENTATION_PLAN.md; docs/adr/**; this plan |
| Agent 1: Platform / Engine | core/common/**; core/model/**; core/browser/**; engine/gecko/**; benchmark/** |
| Agent 2: Product UI | design-system/**; feature/** |
| Agent 3: Data / Extension Trust / Release | core/data/**; core/extensions/**; testing/web-fixtures/**; .github/**; scripts/ci/**; scripts/release/**; docs/release/**; docs/security/**; docs/legal/**; gradle/verification-metadata.xml; LICENSE*; NOTICE*; THIRD_PARTY_NOTICES* |

- Module-local build.gradle.kts, manifests, resources, source, and tests belong to the module owner.
- Root dependency and plugin versions remain lead-only even when requested by a module owner.
- Application navigation, dependency injection, and cross-module wiring under app/** remain lead-only.
- A worker needing a lead-only edit submits the requested path, exact change, reason, and consuming commit SHA.
- A worker needing another owner's change submits a contract request and continues only on independent work.

## Module-local test ownership

- Agent 1 owns src/test/**, src/androidTest/**, and src/testFixtures/** inside its platform and engine modules.
- Agent 2 owns module-local unit, instrumentation, screenshot, semantics, and focus tests inside design-system/** and feature/**.
- Agent 3 owns module-local migration, extension security, fixture, unit, and instrumentation tests inside its modules.
- Agent 3 owns the shared implementation of :testing:web-fixtures, but each consumer owns the tests that use it in that consumer's module.
- The lead owns integration tests under app/src/androidTest/** after the required worker commits exist.
- No agent fixes another module's failing test by editing that module. The failure returns to its owner.

## Immutable base and contract-freeze gate

Parallel work starts only after the lead records one immutable base commit for the wave.

- Serialized bootstrap imports upstream history, preserves licensing, establishes module shells, and fixes the ownership map.
- Browser state, commands, events, IDs, repository ports, extension trust types, fixture APIs, UI state boundaries, and dependency directions are reviewed before consumer implementation.
- The lead records accepted contracts in the roadmap or an ADR and commits all lead-owned wiring required by the wave.
- All three worker branches and worktrees start from the exact same base SHA.
- The lead makes no commit on the integration base while workers are active.
- Workers do not change frozen cross-module contracts during the wave.
- Non-blocking contract changes wait for the next wave.
- A blocking contract defect stops integration. The lead closes the wave, creates a corrected base, and recreates affected worker branches from it.

## Branch and worktree convention

| Purpose | Pattern | Example |
|---|---|---|
| Integration base | integration/wave-NN-slug | integration/wave-02-one-tab |
| Agent 1 | agent/platform/wave-NN-slug | agent/platform/wave-02-one-tab |
| Agent 2 | agent/product/wave-NN-slug | agent/product/wave-02-one-tab |
| Agent 3 | agent/data-trust/wave-NN-slug | agent/data-trust/wave-02-one-tab |

The repository root remains the lead's integration worktree. Worker locations are .worktrees/platform, .worktrees/product, and .worktrees/data-trust.

After bootstrap and contract freeze, the lead creates an example wave with PowerShell-compatible commands:

~~~powershell
rtk git switch main
rtk git switch -c integration/wave-02-one-tab
rtk powershell.exe -NoProfile -Command "New-Item -ItemType Directory -Force -Path '.worktrees' | Out-Null"
rtk git worktree add .worktrees/platform -b agent/platform/wave-02-one-tab integration/wave-02-one-tab
rtk git worktree add .worktrees/product -b agent/product/wave-02-one-tab integration/wave-02-one-tab
rtk git worktree add .worktrees/data-trust -b agent/data-trust/wave-02-one-tab integration/wave-02-one-tab
rtk git rev-parse integration/wave-02-one-tab
rtk git -C .worktrees/platform rev-parse HEAD
rtk git -C .worktrees/product rev-parse HEAD
rtk git -C .worktrees/data-trust rev-parse HEAD
~~~

The four reported SHAs must match before any worker edits a file.

## Worker staging and commit rules

- Concurrent commits are allowed only in the three separate worktrees.
- Two agents never use the same worktree or branch.
- git add ., git add -A, root-wide wildcards, and git commit -a are forbidden.
- Workers stage explicit owned paths with rtk git add -- <owned-paths>.
- Workers inspect rtk git diff --cached --name-only before every commit.
- Workers do not run merge, rebase, pull, push, cherry-pick, or integration commands.
- Workers do not rewrite history after reporting a commit SHA.
- A worker may make focused follow-up commits on its branch when the lead returns an ownership or contract issue.
- Each worker reports base SHA, branch, final commit SHA, changed paths, tests run, and unrun checks.

Example path-specific commits:

~~~powershell
rtk git -C .worktrees/platform add -- core/common core/model core/browser engine/gecko benchmark
rtk git -C .worktrees/platform diff --cached --name-only
rtk git -C .worktrees/platform commit -m "feat(platform): implement wave 02 engine slice"

rtk git -C .worktrees/product add -- design-system feature
rtk git -C .worktrees/product diff --cached --name-only
rtk git -C .worktrees/product commit -m "feat(product): implement wave 02 browser UI"

rtk git -C .worktrees/data-trust add -- core/data core/extensions testing/web-fixtures .github scripts/ci scripts/release docs/release docs/security docs/legal gradle/verification-metadata.xml
rtk git -C .worktrees/data-trust diff --cached --name-only
rtk git -C .worktrees/data-trust commit -m "feat(data-trust): implement wave 02 persistence and fixtures"
~~~

## Changed-path overlap preflight

The lead runs this after all three final SHAs are reported and before tests or merge:

~~~powershell
rtk git diff --name-only integration/wave-02-one-tab...agent/platform/wave-02-one-tab
rtk git diff --name-only integration/wave-02-one-tab...agent/product/wave-02-one-tab
rtk git diff --name-only integration/wave-02-one-tab...agent/data-trust/wave-02-one-tab
rtk powershell.exe -NoProfile -Command "$base='integration/wave-02-one-tab'; $branches=@('agent/platform/wave-02-one-tab','agent/product/wave-02-one-tab','agent/data-trust/wave-02-one-tab'); $seen=@{}; foreach ($branch in $branches) { $range=$base+'...'+$branch; foreach ($path in (& rtk git diff --name-only $range)) { if ($seen.ContainsKey($path)) { Write-Error ('changed-path overlap: '+$path+' in '+$seen[$path]+' and '+$branch); exit 1 }; $seen[$path]=$branch } }"
~~~

- Every changed path must match exactly one ownership row.
- Any overlap or out-of-scope path rejects the branch from integration.
- The lead does not resolve overlap by editing the integration branch.
- The issue returns to owner branches for corrective commits, then the full preflight runs again.
- Clean path separation does not replace contract, security, accessibility, or lifecycle review.

## Lead-only integration and octopus merge

After preflight, focused checks, the repository quality gate, and required specialist reviews, the lead creates one octopus merge commit containing the three worker heads:

~~~powershell
rtk git switch integration/wave-02-one-tab
rtk git merge --no-ff --no-edit agent/platform/wave-02-one-tab agent/product/wave-02-one-tab agent/data-trust/wave-02-one-tab
~~~

- The merge must have the integration base plus all three worker heads as parents.
- The lead does not cherry-pick worker commits or create three sequential integration merges.
- If Git reports a conflict, the lead runs rtk git merge --abort and returns the conflict to responsible owner branches.
- Owners correct the issue with new commits on their branches without merging or rebasing another worker branch.
- The lead repeats preflight, checks, and the octopus merge after corrected SHAs are reported.
- If main must show one linear commit, retain octopus history on the integration branch and use squash-at-PR merge. Do not destroy the worker and integration audit trail before review.

## Dependency-ordered execution waves

| Wave | Lead / Integration Captain | Agent 1: Platform / Engine | Agent 2: Product UI | Agent 3: Data / Extension Trust / Release | Gate |
|---|---|---|---|---|---|
| 0. Serialized import and governance | Import/tag upstream, preserve licensing, establish root build and ownership skeleton | Inventory engine removal and Gecko migration | Inventory identity, UI, focus, and accessibility | Resolve license, provenance, trust, CI, and release requirements | No parallel writes; accepted bootstrap commit exists |
| 1. Contract definition | Create module shells, root dependencies, fake wiring, and integrate contract branches | Define browser models, commands, events, IDs, lifecycle, and engine ports | Define tokens, focus primitives, screen-state inputs, and fake UI boundaries | Define repository ports, extension trust types, persistence semantics, and fixture API | Lead records contract-freeze commit |
| 2. One-tab vertical slice | Create worktrees from frozen SHA and integrate worker heads | Implement runtime, session, delegates, attachment, navigation, and recreation | Implement browser host, omnibox, controls, progress, errors, and deterministic focus | Implement initial persistence, fixture server, CI slice, and extension domain stubs | One octopus merge; controlled tab survives recreation |
| 3. Tabs, persistence, and shell | Wire modules in :app and integrate wave | Implement tab reducer, session lifecycle, suspension, crash recovery, and memory policy | Implement design system, adaptive shell, tab switcher, private state, and focus restoration | Implement Room tab records, migrations, session references, and private-data separation | Restore, privacy, lifecycle, and focus criteria pass |
| 4. Browser and local-data completeness | Integrate navigation and composition | Implement prompts, permissions, security, media, uploads, downloads, and external intents | Implement home, bookmarks, history, downloads, settings, About, and clear-data UI | Implement repositories, import/export, retention, site policy, records, and fixtures | Browser and local-data matrices pass |
| 5. WebExtensions | Integrate engine, trust, persistence, and UI contracts | Implement Gecko extension adapter, action routing, and crash recovery | Implement install review, manager, details, popup, options, badges, compatibility, and private-access UI | Implement signed-source validation, permissions, updates, storage, native-messaging allowlist, threat tests, and recovery | No unresolved critical or high security finding |
| 6. Hardening and release | Integrate final wiring, run full gate, and prepare release PR | Optimize startup, switching, memory, engine compatibility, ABI, and size | Complete accessibility, localization, RTL, reduced motion, overscan, and TV polish | Complete CI matrix, signing, SBOM, notices, provenance, rollout, rollback, and playbooks | Reproducible release and v1 matrix pass |

Each parallel wave repeats: freeze base, create worktrees, commit independently, preflight paths, run checks, obtain specialist review, and create one lead-owned octopus merge commit.

## Required handoff contracts

| Contract | Producer | Consumers | Required handoff |
|---|---|---|---|
| Browser state, commands, events, IDs, lifecycle, and engine ports | Agent 1 | Agent 2, Agent 3, lead | Public symbols, invariants, failures, fake implementation, and tests |
| Design tokens, focus primitives, screen-state needs, and UI events | Agent 2 | Agent 1, Agent 3, lead | Component API, semantics, focus order, Back behavior, and sample state |
| Repository ports, persistence semantics, migrations, and clear-data behavior | Agent 3 | Agent 1, Agent 2, lead | Interfaces, transactions, private-mode rules, and fixtures |
| Extension trust, permissions, compatibility, private access, and updates | Agent 3 | Agent 1, Agent 2, lead | Typed states, decision table, validation failures, consent rules, and threats |
| Gecko extension adapter behavior | Agent 1 | Agent 2, Agent 3, lead | Operations, event mapping, lifecycle limits, and error mapping |
| Deterministic fixture API | Agent 3 | Agent 1, Agent 2, lead | Endpoint catalog, certificates, startup/cleanup contract, and examples |
| Application wiring request | Any worker | Lead | Exact lead path, requested binding, reason, and worker commit SHA |

- Handoffs are contract artifacts, not permission to edit another owner's path.
- Consumers use frozen interfaces or fakes and do not copy implementation across modules.
- The lead reconciles disagreements before creating the next base.

## First assignments

| Order | Owner | Assignment | Completion evidence |
|---|---|---|---|
| 1 | Lead | Import/tag TV Bro baseline, establish root build policy, add module shells, and record ADR-0004 | Bootstrap commit and baseline tag |
| 2 | Agent 1 | Add :core:common, :core:model, and :core:browser contracts, fake engine behavior, and contract tests | Platform SHA and owned-path list |
| 3 | Agent 2 | Add design tokens, focus primitives, browser screen-state contract, fake one-tab UI, and tests | Product SHA and owned-path list |
| 4 | Agent 3 | Add repository ports, :core:extensions trust types, :testing:web-fixtures API, and contract tests | Data-trust SHA and owned-path list |
| 5 | Lead | Run path preflight, checks, and specialist reviews, then create the contract-wave octopus merge | One merge commit with three worker parents |
| 6 | Lead | Freeze merged contracts and create Wave 2 worktrees from the exact SHA | Matching SHA in all worktrees |
| 7 | Agent 1 | Implement the one-tab GeckoView engine slice | Platform vertical-slice commit |
| 8 | Agent 2 | Implement the one-tab Compose browser against frozen contracts | Product vertical-slice commit |
| 9 | Agent 3 | Implement initial persistence, fixtures, CI slice, and trust stubs | Data-trust vertical-slice commit |
| 10 | Lead | Integrate worker heads, application wiring, and acceptance tests | One octopus merge plus explicit lead wiring commit if needed |

## Definition of done

- Requested behavior and failure states close roadmap acceptance criteria.
- Each worker changed only exclusive paths.
- Implementation and module-local tests are committed together.
- All workers started from the recorded immutable base SHA.
- Changed-path preflight reports no overlap or unowned path.
- Frozen contracts and dependency directions remain intact.
- Required GeckoView, Material 3 TV, extension security, privacy, licensing, and release reviews are complete.
- Formatting, static analysis, unit, focused instrumentation, and packaging checks pass, or exact unrun checks and reasons are recorded.
- D-pad, Back, keyboard, mouse, touch, accessibility, private mode, and process recreation are covered where applicable.
- No WebView/Blink fallback, unsigned production extension path, secret, browsing data, or hidden network dependency is introduced.
- The lead integrates the three worker heads with one octopus merge and records lead-only wiring explicitly.
- The wave summary lists decisions, SHAs, tests, residual risks, and follow-up work.

## Load balancing

- Load moves only at a wave boundary, never while worker branches are active.
- Rebalancing transfers an entire module or path prefix and is recorded before the next base is frozen.
- Agent 3 retains extension trust policy, Room migrations, signing, provenance, and private-data rules even when overloaded.
- Agent 2 retains feature/extensions UI while Agent 3 retains :core:extensions trust logic.
- Agent 1 may absorb :testing:web-fixtures or CI mechanics in a later wave only after ownership is updated and a new base is created.
- The lead may absorb shared integration automation, but not feature implementation.
- No load-balancing decision creates two writers for the same path.
