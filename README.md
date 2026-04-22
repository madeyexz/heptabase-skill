# heptabase-skill

Skill and tooling for creating real cross-links between [Heptabase](https://heptabase.com) cards via the `heptabase` CLI.

The key insight: markdown wiki-link syntax (`[[Card Title]]`) is stored as literal text, not a live link. Real links require `heptabase note save` with a ProseMirror `card` node. This repo captures the schema and automates the workflow.

## Files

| Path | What it is |
|---|---|
| [`skill.md`](./skill.md) | The full skill doc — ProseMirror `card` node schema, the end-to-end workflow, gotchas, and the recipe for rediscovering schemas for other node types (embeds, mentions, etc.). Start here. |
| [`bin/heptabase-link`](./bin/heptabase-link) | Bash + `jq` script that automates the read → merge → save → verify flow between two existing cards. Bidirectional by default, idempotent, uses `contentMd5` for conflict detection. |

## Quickstart

```bash
# Prereqs: heptabase CLI in PATH (the desktop app must be running), jq installed.

# Link two existing cards (bidirectional):
./bin/heptabase-link <card-id-a> <card-id-b>

# One-way only, custom label text:
./bin/heptabase-link <card-id-a> <card-id-b> --one-way --label "Related"
```

See [`skill.md`](./skill.md) for the full schema and workflow rationale.
