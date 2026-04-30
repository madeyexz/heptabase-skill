# Contributing

## Skills in this repo

- `skills/heptabase-cli` — copied verbatim from [heptameta/heptabase-cli-skills](https://github.com/heptameta/heptabase-cli-skills) (MIT). When upstream releases a new version, sync the file and bump the plugin version. Do not edit it locally except to resolve sync conflicts.
- `skills/heptabase-linking` — maintained in this repo.

## Release Process

Release a new version when either skill's behavior or interface changes — e.g. a sync from upstream `heptabase-cli`, new recipes in `heptabase-linking`, or breaking changes to `heptabase-link`.

Claude Code caches installed plugins by resolved plugin version, so users only receive updates when the plugin version changes. Pushing new commits without a version bump is not enough for installed users.

Claude Code resolves the plugin version in this order:

1. `.claude-plugin/plugin.json` `version`
2. `.claude-plugin/marketplace.json` plugin entry `version`
3. Git commit SHA

`.claude-plugin/plugin.json` wins, so keep the plugin version there as the single source of truth. Do not add a plugin entry `version` in `.claude-plugin/marketplace.json` unless we intentionally change the versioning strategy.

## Version Bump Guidelines

- Patch: documentation-only fixes, typos, clarifications.
- Minor: new recipes, new gotchas, sync from upstream `heptabase-cli`, non-breaking changes to `heptabase-link`.
- Major: breaking changes to the script's CLI, removal/rename of a skill.

## Release Checklist

- Bump `.claude-plugin/plugin.json` `version`.
- Validate both skills:
  - `npx --yes skills-ref validate ./skills/heptabase-linking`
  - `npx --yes skills-ref validate ./skills/heptabase-cli`
- Validate the marketplace manifest: `claude plugin validate .`.
- Commit, tag, and push the release.

## Syncing `heptabase-cli` from upstream

```bash
gh api repos/heptameta/heptabase-cli-skills/contents/skills/heptabase-cli/SKILL.md \
  --jq '.content' | base64 -d > skills/heptabase-cli/SKILL.md
```

Diff, sanity-check that the `heptabase-cli-version-range` in the frontmatter still matches the CLI versions you care about, then bump and release.
