---
name: heptabase-cli
description: Interact with Heptabase using the CLI to create, read, and edit notes, journals, tags, cards, list whiteboards and manage cards on whiteboards, and browse AI Tutor goals, courses, and lessons. Use when the user asks to manage their Heptabase knowledge base, search cards, work with journals, tags, or whiteboards, or read AI Tutor content.
allowed-tools: Bash(heptabase *) Bash(jq *)
metadata:
  heptabase-cli-version-range: "0.2.x"
---

## Prerequisites

- CLI installed from the desktop app. The command is `heptabase` on macOS/Linux; Windows installs `heptabase.cmd` for cmd/PowerShell and a `heptabase` shim for POSIX shells.
- Check version compatibility before use with `heptabase --version`. If the installed CLI version is outside this skill's compatibility range (`0.2.x`), you MUST stop and ask the user to update either the Heptabase desktop app or this skill package before continuing.

## Command discovery

Run `heptabase help` to see all available top-level commands. This is always up to date. Each command supports `--help` for detailed usage:

```bash
heptabase help
heptabase note --help
heptabase note create --help
```

## Common recipes

Use these as quick recipes for frequent requests. For less common flags or if a command fails, run `heptabase help` or `<command> --help` to discover the correct syntax.

- **Recent cards:** `heptabase card list --sort createdTime --direction descending --limit 20`
- **Today's journal:** `heptabase journal read $(date +%Y-%m-%d)`
- **Search cards by keyword:** `heptabase card list -q "<keyword>" --limit 20`
- **List cards on a whiteboard:** `heptabase whiteboard cards <whiteboardId>`
- **Add a card to a whiteboard:** `heptabase whiteboard add-card --whiteboard-id <whiteboardId> --card-id <cardIdOrDate>`

## All output is JSON

Every command prints JSON to stdout. You can parse it with `jq` or pipe it to other tools.

## Troubleshooting

- **Desktop app must be running.** The CLI communicates with a local server inside the app. If the app is closed, all commands fail. Run `heptabase start` to launch and wait for readiness.
- **Mutations are serialized.** Write operations (create, save, append, trash, restore, tag add/remove, whiteboard add-card/remove-card) run one at a time to prevent conflicts. Reads are concurrent.
- **Request body size limit.** The server rejects request bodies larger than 1 MB.
- **Request timeout.** The server times out requests that take longer than 10 seconds to send their body.

## Known limitations

- **Auto-enabling local server/CLI install not supported.** If the local CLI server is disabled or CLI wiring is missing, the skill cannot repair it by itself; ask the user to enable Local CLI Server and CLI install from desktop settings first.
- **Binary/media upload workflows not supported.** This skill is for JSON/text operations on notes/journals/tags/cards and AI Tutor reads, not file upload or media-processing APIs.
- **Whiteboard creation/edit/delete not supported yet.** You can list whiteboards and add, list, or remove cards on them, but you can't create, rename, move, or delete whiteboards.
- **File reading not supported yet.** You can't read files (e.g., image, video) with `fileId`.
- **PDF card reading not supported yet.** You can't read pdf card or its parsed content.

## Warnings

- **Use the CLI as the only data access path.** Never directly read, write, or modify Heptabase app data through local database files, app storage, cache files, internal endpoints, or any other non-CLI mechanism. If the CLI does not support the requested operation, stop and report that it is not supported.
