# tw-worga

Workday (`wor`) + organisational (`ga`) workflows layered on top of [`rimir/cascade-palette`](https://github.com/rimir-cc/tw-cascade-palette) and [`rimir/kind`](https://github.com/rimir-cc/tw-kind). Demonstrates two integration patterns: (1) cp's sticky-context mechanism — pin people during a meeting, then watch context-aware views narrow to topics / tasks / notes involving the pinned set; and (2) conversation-centric verbs over a kind data model — take a call (creates a running conversation and drills into it), then surface and create the work items / topics in the context of its participants.

The plugin is intentionally small. It demonstrates the integration pattern; bring your own data model (kind, orga-data-model, hand-rolled tags) and adapt the filters via the three shipped config tiddlers to match your fields.

## Key features

- **`Take call from this person`** action — Space on a person row fires the verb: **creates a running `conversation`** (a `rimir/kind` instance) parented to the person, with that person as participant, `planned-at: now` and a `Conversation at <date time>` caption, then **drills into the new conversation** — its action menu on top, and popping out reveals the person's *Referenced by…* conversation list, then wherever you came from. Palette stays open (`ca-after-fire: keep`). Uses cp's `drill-sequence` message + kind's reverse-nav + create procedures. Surfaces on rows with `kind.type: person` AND on any tiddler tagged `Person`.
- **`In context of participants…`** action — on a `conversation` row, drills into the open work items and topics referencing the conversation's participants (teams expanded to member persons). Projects / tasks / todos clustered by role — *Waiting for*, *Assignee*, *Stakeholder* — and stakeholder *Topics* in their own cluster, each sorted soonest-actionable first (done / archived excluded). The same drill carries **New Project / Task / Todo / Topic…** rows: type a caption and ↵ to create one parented to the conversation, with the participants as stakeholders — it reappears in its cluster on the spot.
- **`Start meeting`** leader — bound to `m`. Clears the sticky context and opens the *Pick attendees* view.
- **`Pick attendees`** view — checkbox-style multiselect. Each row carries ☑/☐ mirroring whether that person is currently in sticky context. Enter toggles in/out. Pinned rows float to the top. Esc closes the picker; the populated context survives.
- **`Topics for context`** view — worked example of a context-aware view. Narrows to topics whose `stakeholders` field contains at least one pinned tiddler. Declares `ca-view-context-aware: yes` so the view pill paints an indigo badge when context is non-empty. Override `$:/config/rimir/worga/topics-stakeholder-field` to match your data model.

## Prerequisites

- TiddlyWiki ≥ 5.4.0
- [`rimir/cascade-palette`](https://github.com/rimir-cc/tw-cascade-palette) ≥ 0.0.124 — provides sticky-context + the view machinery as before (0.0.96–0.0.98), plus the `rimir-cascade-palette-drill-sequence` message (0.0.124) that `Take call` uses to create-then-land-in-context. Without 0.0.124 the `Take call` landing falls back to a no-op (the conversation is still created).
- [`rimir/kind`](https://github.com/rimir-cc/tw-kind) ≥ 0.1.70 — **required** for `Take call` and `In context of participants…` (they create / list `kind` instances and reuse kind's reverse-nav + create procedures). A `conversation` type plus `project` / `task` / `todo` / `topic` / `person` / `team` types with the expected fields (`participants`, `parent`, `stakeholder`, `assignee`, `waiting-for`, `status`) must exist in your wiki. The `Pick attendees` / `Topics for context` flows still work tag-based without kind.

## Quick start

1. Install `rimir/cascade-palette` 0.0.124+, `rimir/kind` 0.1.70+ and this plugin. Restart.
2. Open the palette (Ctrl-Space). The Context pill strip is empty.
3. On any person row, Space → *Take call from this person*. A running `conversation` is created parented to the person, and the palette drills into it: you land on the conversation's action menu. From there, drill *In context of participants…* to see (or inline-create) the work items / topics involving the participants. Press **Esc** to pop back to the person's conversation list, then to where you came from.
5. For a multi-person meeting, type `m` (leader) → *Pick attendees* view opens. Enter on each attendee toggles in/out of context; view stays open; ☑/☐ updates live; pinned rows float to the top. **Esc** returns to the view you were in when you pressed `m` (the picked attendees stay in sticky context).
6. Shift-`C` clears the sticky context.

## Configuration

Three shadow config tiddlers let you adapt to your data model without forking:

- `$:/config/rimir/worga/topics-stakeholder-field` — default `stakeholders`. The field *Topics for context* reads to find pinned people. If your topics use `stakeholder` (singular), `assignee`, `owner`, etc., change it here.
- `$:/config/rimir/worga/topics-roots-filter` — default `[all[tiddlers]has[stakeholders]]`. Base "topics" candidates. If you have a typed model, scope to it: e.g. `[all[tiddlers]field:kind.type[topic]]`.
- `$:/config/rimir/worga/attendees-roots-filter` — default unions kind-typed and tag-typed persons, both with `!is[system]` so kind's self-describing type-definition tiddlers stay out of the picker.

## Adding your own context-aware view

```
title: $:/your/views/whatever
tags: $:/tags/rimir/cascade-palette/view
ca-view-name: My narrowed view
ca-view-context-aware: yes
ca-view-roots: [all[tiddlers]has[<your-field>]] :filter[<currentTiddler>get[<your-field>]enlist-input[] :intersection[enlist<sticky-context-list>]]
```

The badge appears when sticky context is non-empty AND `ca-view-context-aware: yes`. The narrowing happens in the filter — there's no engine-level auto-narrow.

## Adding your own toggle-multiselect

For a checkbox-style multiselect view (the pattern *Pick attendees* uses):

```
title: $:/your/views/multiselect
tags: $:/tags/rimir/cascade-palette/view
ca-view-name: Pick X
ca-view-roots: [all[tiddlers]field:kind.type[YOUR-TYPE]!is[system]]
ca-view-sort: custom
ca-view-sort-key: [enlist<sticky-context-list>match<currentTiddler>then[0]else[1]addsuffix<currentTiddler>]
ca-view-row-icon: [enlist<sticky-context-list>match<currentTiddler>then[☑]else[☐]]
ca-view-after-fire: stay
ca-view-row-actions: <$action-sendmessage $message={{{ [enlist<sticky-context-list>match<currentTiddler>then[rimir-cascade-palette-unpin-context]else[rimir-cascade-palette-pin-context]] }}} title=<<currentTiddler>>/>
```

Two patterns coordinate: the row-icon + sort-key give visual state; the inline-filter `$message` toggles per row. Enter on any row pins or unpins; the palette stays open between toggles thanks to `ca-view-after-fire: stay`.

## License

MIT — see `LICENSE.md`.
