# heptabase-skill

Tools for linking [Heptabase](https://heptabase.com) cards together using the `heptabase` command line.

If you type `[[Card Title]]` into a card through the CLI, Heptabase shows it as plain text — not a clickable link. Making a real link needs a specific piece of JSON. This repo explains what that is, and gives you a script that does it for you.

This repo is a [skills.sh](https://skills.sh) skill — the layout follows the standard `skills/<name>/SKILL.md` convention.

## Install

```bash
npx skills add madeyexz/heptabase-skill
```

## Files

| Path | What it is |
|---|---|
| [`skills/heptabase-linking/SKILL.md`](./skills/heptabase-linking/SKILL.md) | The skill itself. Notes on how Heptabase stores card links, step-by-step instructions for making one from the command line, and the mistakes to avoid. Read this first. |
| [`skills/heptabase-linking/bin/heptabase-link`](./skills/heptabase-linking/bin/heptabase-link) | A script that links two cards for you. Runs both ways by default. Safe to run twice — it won't add duplicates. |

## Quickstart

You need the Heptabase desktop app running with the CLI turned on, and `jq` installed.

```bash
# Link two cards, both ways:
./skills/heptabase-linking/bin/heptabase-link <card-id-a> <card-id-b>

# One way only, with a custom label:
./skills/heptabase-linking/bin/heptabase-link <card-id-a> <card-id-b> --one-way --label "Related"
```

See [`SKILL.md`](./skills/heptabase-linking/SKILL.md) for the full story.
