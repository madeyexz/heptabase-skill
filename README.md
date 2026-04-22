# heptabase-skill

Tools for linking [Heptabase](https://heptabase.com) cards together using the `heptabase` command line.

If you type `[[Card Title]]` into a card through the CLI, Heptabase shows it as plain text — not a clickable link. Making a real link needs a specific piece of JSON. This repo explains what that is, and gives you a script that does it for you.

## Files

| Path | What it is |
|---|---|
| [`skill.md`](./skill.md) | Notes on how Heptabase stores card links, step-by-step instructions for making one from the command line, and the mistakes to avoid. Read this first. |
| [`bin/heptabase-link`](./bin/heptabase-link) | A script that links two cards for you. Runs both ways by default. Safe to run twice — it won't add duplicates. |

## Quickstart

You need the Heptabase desktop app running with the CLI turned on, and `jq` installed.

```bash
# Link two cards, both ways:
./bin/heptabase-link <card-id-a> <card-id-b>

# One way only, with a custom label:
./bin/heptabase-link <card-id-a> <card-id-b> --one-way --label "Related"
```

See [`skill.md`](./skill.md) for the full story.
