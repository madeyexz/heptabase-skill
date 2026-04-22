---
name: heptabase-linking
description: "Create real cross-links between Heptabase cards via the heptabase CLI. Use when the user wants to link Heptabase cards, cross-reference notes, build a card graph, or fix existing `[[Card Title]]` wiki-link text that is rendering as plain text instead of a clickable card pill. Real links require a ProseMirror `card` node written through `heptabase note save`; markdown paths (`note create`, `note append`) cannot create them. Triggers on: heptabase link cards, cross-link heptabase, heptabase cross-reference, heptabase cardId node, heptabase note save, heptabase prosemirror link, heptabase wiki link not working."
---

# Heptabase card cross-linking via CLI

## The key finding

**Markdown append does NOT create real cross-links in Heptabase.** Wiki-link syntax like `[[Card Title]]` written through `heptabase note create` or `heptabase note append` is stored as literal text inside a `text` node — it renders in the card as the plain string `[[Card Title]]`, not as a clickable link.

Real internal links are dedicated ProseMirror **node types** (not marks). The only way to create them via the CLI is `heptabase note save` (or `journal save`) with explicit ProseMirror JSON.

## Two link schemas: pick the right one for your target

Heptabase uses **different node types** depending on whether you're linking to a card (note, pdf, highlight, etc.) or to a journal. Getting this wrong produces a pill that renders as "Invalid card" in the UI.

### Linking to a card (note, pdf, highlight, etc.)

```json
{ "type": "card", "attrs": { "cardId": "<target-card-uuid>" } }
```

- `cardId` is the target's UUID
- Works for any non-journal object type

### Linking to a journal

```json
{ "type": "date", "attrs": { "date": "2026-03-12" } }
```

- It's a `date` node, **not** a `card` node
- The attribute is `date` (the YYYY-MM-DD string), **not** `cardId`
- Using `{type:"card", attrs:{cardId:"2026-03-12"}}` saves without error but renders as "Invalid card"

### Common properties

- **Both are nodes, not marks.** `link` (external URL) is a mark that wraps text. `card` and `date` are standalone inline nodes — they sit inside a paragraph alongside text nodes, no wrapped content.
- **Inline level.** They must live inside a block node like `paragraph`. Do not put them at the top level of the `doc`.
- **No label.** Heptabase resolves the display text from the target's current title (for cards) or date (for journals) at render time.
- **Not to be confused with external links.** External URLs are a `link` mark on a `text` node:
  ```json
  { "type": "text", "marks": [{ "type": "link", "attrs": { "href": "https://…", "data-internal-href": null } }], "text": "…" }
  ```
  The `data-internal-href: null` field is a telltale that external and internal links share a schema lineage, but **do not** try to create internal links by setting `data-internal-href` — internal links go through the `card` or `date` nodes instead.

## Minimum working example

A paragraph containing "See also: " followed by a live link to another card:

```json
{
  "type": "paragraph",
  "content": [
    { "type": "text", "text": "See also: " },
    { "type": "card", "attrs": { "cardId": "514661f9-10f3-4305-b58f-07e552103021" } }
  ]
}
```

Wrapped in a full doc:

```json
{
  "type": "doc",
  "content": [
    {
      "type": "heading",
      "attrs": { "level": 1 },
      "content": [{ "type": "text", "text": "Test Card Alpha" }]
    },
    {
      "type": "paragraph",
      "content": [{ "type": "text", "text": "Intro paragraph." }]
    },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "See also: " },
        { "type": "card", "attrs": { "cardId": "<beta-uuid>" } }
      ]
    }
  ]
}
```

You do NOT need to provide `attrs.id` on headings/paragraphs on save — Heptabase will generate those UUIDs for you. Compare before/after with `note read` to confirm.

## End-to-end workflow: cross-link two new cards

The clean recipe. Each step is explicit about why.

### 1. Create both cards first (markdown path is fine for initial creation)

```bash
heptabase note create -c "# Card A

Body text."
# → { "id": "<a-uuid>", "title": "Card A" }

heptabase note create -c "# Card B

Body text."
# → { "id": "<b-uuid>", "title": "Card B" }
```

Why create both first: the `card` node needs a concrete `cardId`, so both targets must exist before you can link to them.

### 2. Read each card to get current `contentMd5`

```bash
heptabase note read <a-uuid>
# → { ..., "contentMd5": "<md5-a>" }

heptabase note read <b-uuid>
# → { ..., "contentMd5": "<md5-b>" }
```

Why: `note save` accepts `--content-md5` for optimistic concurrency. If you skip it and someone else edited the card between your read and write, you'd silently overwrite their changes. Always pass it.

### 3. Build full-replacement ProseMirror JSON for each card

Write the JSON to a file (easier than inline for anything non-trivial — avoids shell quoting hell):

```bash
cat > /tmp/card-a.json <<'EOF'
{
  "type": "doc",
  "content": [
    {
      "type": "heading",
      "attrs": { "level": 1 },
      "content": [{ "type": "text", "text": "Card A" }]
    },
    {
      "type": "paragraph",
      "content": [{ "type": "text", "text": "Body text." }]
    },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "See also: " },
        { "type": "card", "attrs": { "cardId": "<b-uuid>" } }
      ]
    }
  ]
}
EOF
```

Why full replacement: `note save` overwrites the entire card body — there is no "insert node at position" API. You must rebuild the full doc, preserving whatever existing content you want to keep.

### 4. Save each card

```bash
heptabase note save <a-uuid> --content-md5 <md5-a> -f /tmp/card-a.json
heptabase note save <b-uuid> --content-md5 <md5-b> -f /tmp/card-b.json
```

### 5. Verify

```bash
heptabase note read <a-uuid>
```

In the returned `content` string (which is itself JSON-encoded), look for:

```
"type":"card","attrs":{"cardId":"<b-uuid>"}
```

Be aware of **double-encoding**: the outer response has a `content` field whose *value* is a JSON string. `grep` on the raw CLI output will see escaped quotes (`\"type\":\"card\"`). If your grep comes back empty, inspect the raw output first — the link is probably there.

In the desktop app, refresh the card; the link should render as a clickable card pill showing the target's title.

## Gotchas and pitfalls

1. **`[[Title]]` in markdown is dead text.** If you see literal `[[...]]` rendering in a card, it was written via `note create`/`note append` and needs to be rewritten with `note save`.
2. **`note append` also uses the markdown parser.** So you cannot "just append a link" to an existing card — you have to `note read` to get current content, merge a new paragraph with a `card` node, and `note save` the whole thing back.
3. **Cross-links are by ID, not title.** If you rename a target card, existing links keep working (they store the UUID). But this also means you cannot pre-create a link to a card that does not yet exist — the `cardId` must be real.
4. **No dangling links.** The CLI does not verify the `cardId` exists at save time in a way that surfaces nicely — if you save a bogus UUID, the card node persists but renders as a broken link. Always read the target card first to confirm its ID.
5. **External URL links are a different mechanism.** Those are `link` marks on text. Do not mix them up.
6. **Journals use a different node type.** Links to journals are `{type:"date", attrs:{date:"..."}}`, not `{type:"card", attrs:{cardId:"..."}}`. A card-node with a date string in `cardId` saves without error but renders as "Invalid card" in the UI. Verify by opening the UI, not just reading the JSON.
7. **Other card types (pdf, highlightElement, etc.)** likely use the same `card` node schema since the `cardId` field is type-agnostic for card-like objects — but this has only been verified for note→note links. Test before assuming.
8. **`note save` / `journal save` replace everything.** They are not additive. If you forget to include existing paragraphs in your new JSON, they are gone.
9. **Verify renders in the UI, not just the JSON.** A link node can be structurally correct (matches the schema) yet point at something that doesn't exist or uses the wrong node type — and still round-trip through `save` + `read` cleanly. The only ground truth is opening the card in the Heptabase desktop app.

## How this schema was discovered

In case you need to rediscover schemas for other node types (embeds, mentions, block references, etc.):

1. List cards likely to contain the feature (e.g., answer/summary cards often link to source cards).
2. `heptabase note read <cardId>` on candidates.
3. Look for unfamiliar `"type":"..."` values in the ProseMirror JSON `content` field.
4. External URL links appear as `{"type":"text","marks":[{"type":"link","attrs":{"href":"...","data-internal-href":null,...}}],"text":"..."}`.
5. Internal card links appear as `{"type":"card","attrs":{"cardId":"..."}}`.
6. Journal links appear as `{"type":"date","attrs":{"date":"YYYY-MM-DD"}}`.
7. To enumerate every node type in use, sample ~50 notes and journals and collect distinct `.type` values — then inspect each unfamiliar type for its schema:
   ```bash
   for cid in $(heptabase card list --card-types note -l 60 | jq -r '.results[].id'); do
     heptabase note read "$cid" | jq -r '.content // empty' \
       | jq -r '[.. | objects? | .type // empty] | unique | .[]'
   done | sort -u
   ```
8. Other features (embeds, images, etc.) likely follow the same node-vs-mark pattern: standalone visual elements are nodes; text styling/attribution is marks.
