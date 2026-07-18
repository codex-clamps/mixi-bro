# Apply this bootstrap to `codex-clamps/mixi-bro`

The recommended first push preserves TV Bro's Git history and then adds these planning and agent files.

## Recommended: import upstream history first

```powershell
rtk git clone https://github.com/truefedex/tv-bro.git mixi-bro
Set-Location mixi-bro
rtk git checkout 10b8b1a9ff29fcbe5c55052eec61caf553d6964f
rtk git switch -c main
rtk git tag upstream-tv-bro-2.1.6 10b8b1a9ff29fcbe5c55052eec61caf553d6964f
rtk git remote rename origin upstream
rtk git remote add origin https://github.com/codex-clamps/mixi-bro.git
```

Extract the bootstrap archive elsewhere, then copy its files, including the hidden `.codex` directory, into the clone:

```powershell
rtk powershell.exe -NoProfile -Command "Get-ChildItem -LiteralPath 'C:\path\to\mixi-bro-bootstrap' -Force | Where-Object Name -ne 'APPLY_TO_REPOSITORY.md' | Copy-Item -Destination '.' -Recurse -Force"
rtk git add -- AGENTS.md README.md .gitignore .codex docs
rtk git commit -m "Bootstrap mixi-bro plan and multi-agent workflow"
rtk git push -u origin main
rtk git push origin upstream-tv-bro-2.1.6
```

Then protect `main` and perform implementation work on feature branches and draft pull requests.

## Alternative: planning-only bootstrap

For a repository that should remain documentation-only until the import strategy is approved:

```powershell
rtk git clone https://github.com/codex-clamps/mixi-bro.git
Set-Location mixi-bro
rtk powershell.exe -NoProfile -Command "Get-ChildItem -LiteralPath 'C:\path\to\mixi-bro-bootstrap' -Force | Copy-Item -Destination '.' -Recurse -Force"
rtk git add -- AGENTS.md README.md .gitignore .codex docs NOTICE.md licenses
rtk git commit -m "Bootstrap mixi-bro plan and multi-agent workflow"
rtk git push -u origin main
```

When importing TV Bro later, do not overwrite provenance. Record the exact upstream commit/tag and retain its license and copyright.

## Create the three worker worktrees

Run these PowerShell commands from a clean lead checkout after the baseline commit exists. All three branches start from the same captured SHA; record that SHA in every worker assignment and do not advance it during the work wave.

```powershell
$BaseSha = (rtk git rev-parse HEAD).Trim()
rtk git worktree add -b worker/platform-engine .worktrees/platform $BaseSha
rtk git worktree add -b worker/product-ui .worktrees/product $BaseSha
rtk git worktree add -b worker/data-extension-release .worktrees/data-trust $BaseSha
```

Assign the absolute worktree path, branch name, immutable `$BaseSha`, and role allowlist to each worker. Each worker stages explicit owned files only, commits on its assigned branch, and returns its commit SHA. It must never use `git add .`, merge, rebase, pull, or push. After all workers finish, the lead reviews the three path sets and integrates the commits from the lead checkout.

```powershell
rtk git cherry-pick <platform-commit-sha>
rtk git cherry-pick <product-commit-sha>
rtk git cherry-pick <data-trust-commit-sha>
```

## Verify Codex configuration

```powershell
rtk Get-ChildItem .codex -File -Recurse
rtk codex --ask-for-approval never "Summarize the repository instructions and list the available custom agents."
```

Expected custom agent names:

- `architecture_explorer`
- `geckoview_specialist`
- `material3_tv_designer`
- `extension_security_reviewer`
- `test_release_engineer`
- `implementer`
- `platform_engine_implementer`
- `product_ui_implementer`
- `data_extension_release_implementer`