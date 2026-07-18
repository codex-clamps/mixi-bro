# Apply this bootstrap to `codex-clamps/mixi-bro`

The recommended first push preserves TV Bro's Git history and then adds these planning/agent files.

## Recommended: import upstream history first

```bash
git clone https://github.com/truefedex/tv-bro.git mixi-bro
cd mixi-bro
git checkout 10b8b1a9ff29fcbe5c55052eec61caf553d6964f
git switch -c main
git tag upstream-tv-bro-2.1.6 10b8b1a9ff29fcbe5c55052eec61caf553d6964f
git remote rename origin upstream
git remote add origin https://github.com/codex-clamps/mixi-bro.git
```

Extract the bootstrap archive elsewhere, then copy its files—including the hidden `.codex` directory—into the clone:

```bash
rsync -a --exclude APPLY_TO_REPOSITORY.md /path/to/mixi-bro-bootstrap/ ./
git add AGENTS.md README.md .gitignore .codex docs
git commit -m "Bootstrap mixi-bro plan and multi-agent workflow"
git push -u origin main
git push origin upstream-tv-bro-2.1.6
```

Then protect `main` and perform implementation work on feature branches and draft pull requests.

## Alternative: planning-only bootstrap

For a repository that should remain documentation-only until the import strategy is approved:

```bash
git clone https://github.com/codex-clamps/mixi-bro.git
cd mixi-bro
rsync -a /path/to/mixi-bro-bootstrap/ ./
git add .
git commit -m "Bootstrap mixi-bro plan and multi-agent workflow"
git push -u origin main
```

When importing TV Bro later, do not overwrite provenance. Record the exact upstream commit/tag and retain its license and copyright.

## Verify Codex configuration

```bash
find .codex -maxdepth 2 -type f -print | sort
codex --ask-for-approval never "Summarize the repository instructions and list the available custom agents."
```

Expected custom agent names:

- `architecture_explorer`
- `geckoview_specialist`
- `material3_tv_designer`
- `extension_security_reviewer`
- `test_release_engineer`
- `implementer`
