# heptabase-skill

Tools for linking [Heptabase](https://heptabase.com) cards together using the `heptabase` command line.

This repo extends the official [heptameta/heptabase-cli-skills](https://github.com/heptameta/heptabase-cli-skills) plugin (MIT) with a focused skill for **card cross-linking**. Heptabase's `[[Card Title]]` wiki-link syntax doesn't actually create real links when written through the CLI — it stores plain text. Real links require a specific ProseMirror JSON node. This repo documents that schema and ships a script that does it for you.

It bundles the upstream `heptabase-cli` skill verbatim, so installing this plugin gives you both general CLI usage and the cross-linking extension in one place. The layout follows the [skills.sh](https://skills.sh) `skills/<name>/SKILL.md` convention.

## Install

### Claude Code (Marketplace)

```
/plugin marketplace add madeyexz/heptabase-skill
/plugin install heptabase-linking@heptabase-skill
```

### npx skills

```
npx skills add madeyexz/heptabase-skill
```

## Update

For Claude Code marketplace installs:

```
/plugin marketplace update heptabase-skill
/plugin update heptabase-linking@heptabase-skill
/reload-plugins
```

For `npx skills` installs, rerun the install command.

## Skills

This repo bundles two skills:

| Skill | What it covers |
|---|---|
| [`skills/heptabase-cli`](./skills/heptabase-cli/SKILL.md) | General Heptabase CLI usage — command discovery, common recipes, JSON output, troubleshooting, known limitations, and whiteboard card commands. Copied from [heptameta/heptabase-cli-skills](https://github.com/heptameta/heptabase-cli-skills) (MIT). |
| [`skills/heptabase-linking`](./skills/heptabase-linking/SKILL.md) | The specific ProseMirror schema for cross-linking cards — what wiki-link syntax can't do, the `card`/`date` node types, and end-to-end recipes. Ships [`bin/heptabase-link`](./skills/heptabase-linking/bin/heptabase-link), a script that links two cards both ways and is safe to re-run. |

## Quickstart

You need the Heptabase desktop app running with the CLI turned on, and `jq` installed.

```bash
# Link two cards, both ways:
./skills/heptabase-linking/bin/heptabase-link <card-id-a> <card-id-b>

# One way only, with a custom label:
./skills/heptabase-linking/bin/heptabase-link <card-id-a> <card-id-b> --one-way --label "Related"
```

See [`SKILL.md`](./skills/heptabase-linking/SKILL.md) for the full story.
