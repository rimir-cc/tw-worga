# tw-worga

Workday (`wor`) + organisational (`ga`) workflows layered on top of [`rimir/cascade-palette`](https://github.com/rimir-cc/tw-cascade-palette). Three demo verbs and two context-aware views demonstrate how to assemble a personal-workday flow from cp's sticky-context mechanism — pin people during a call or meeting, then watch other views narrow to topics / tasks / notes involving the pinned set.

The plugin is intentionally small. It demonstrates the integration pattern; bring your own data model (kind, orga-data-model, hand-rolled tags) and adapt the filters via the three shipped config tiddlers to match your fields.

## Key features

- **`Take call from <person>`** action — Space on a person row fires `Take call from …`: clears the sticky context, pins the row, AND jumps to the *Topics for context* view. One keystroke = ready to talk. Palette stays open (`ca-after-fire: keep`) so you can follow up immediately (take notes, pin a second person, switch view). Surfaces on rows with `kind.type: person` (when `rimir/kind` is installed) AND on any tiddler tagged `Person` (kind-agnostic fallback).
- **`Start meeting`** leader — bound to `m`. Clears the sticky context and opens the *Pick attendees* view.
- **`Pick attendees`** view — checkbox-style multiselect. Each row carries ☑/☐ mirroring whether that person is currently in sticky context. Enter toggles in/out. Pinned rows float to the top. Esc closes the picker; the populated context survives.
- **`Topics for context`** view — worked example of a context-aware view. Narrows to topics whose `stakeholders` field contains at least one pinned tiddler. Declares `ca-view-context-aware: yes` so the view pill paints an indigo badge when context is non-empty. Override `$:/config/rimir/worga/topics-stakeholder-field` to match your data model.

## Prerequisites

- TiddlyWiki ≥ 5.4.0
- [`rimir/cascade-palette`](https://github.com/rimir-cc/tw-cascade-palette) ≥ 0.0.97 — provides sticky-context, the `ca-view-after-fire: stay` view-level keep-open default (0.0.96), and ambient `<<sticky-context-list>>` in every `_filterInScope` (0.0.97). Without 0.0.97, the ☑/☐ row icon in Pick attendees always shows ☐ regardless of pin state.
- [`rimir/kind`](https://github.com/rimir-cc/tw-kind) is optional — when installed, verbs route via `kind.type`; otherwise they fall back to tag-based matching.

## Quick start

1. Install `rimir/cascade-palette` 0.0.97+ and this plugin. Restart.
2. Open the palette (Ctrl-Space). The Context pill strip is empty.
3. On any person row, Space → *Take call from this person*. Sticky context is cleared, the person is pinned, palette jumps to *Topics for context*.
4. Switch back to your usual view at any time — the Context strip still shows the pinned person until you clear it (Shift-`C` leader) or unpin (Backspace on the pill).
5. For a multi-person meeting, type `m` (leader) → *Pick attendees* view opens. Enter on each attendee toggles in/out of context; view stays open; ☑/☐ updates live; pinned rows float to the top. Esc closes the picker.
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
