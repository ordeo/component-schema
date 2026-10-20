# Ordeo component schema

JSON Schemas for the platform-neutral component descriptions used by Ordeo.

Published at <https://ordeo.github.io/component-schema/>.

A component file describes what the design tokens cannot: which parts a component has, what each
part shows, how the parts nest and how they are placed and laid out. Every colour, size and spacing
stays in the tokens. Tooling reads the file to generate platform code and design-tool components.

## Using a schema

Reference the versioned URL from a component file, and your IDE validates it:

```json
{
  "$schema": "https://ordeo.github.io/component-schema/component/v1.schema.json",
  "components": [
    ...
  ]
}
```

## Files and parts

A file holds one component. `components` lists its parts, the root first. Every part has:

| Key           | Meaning                                                                                  |
|---------------|------------------------------------------------------------------------------------------|
| `name`        | PascalCase, prefixed with the part that holds it: `ButtonLabel` inside `Button`.         |
| `tokens`      | The token paths that style the part, e.g. `component.button.label.base`.                 |
| `content`     | What the part shows. See [Content](#content).                                            |
| `variants`    | The axes the consumer picks. See [Variants and states](#variants-and-states).            |
| `states`      | The interaction states its tokens style.                                                 |
| `placement`   | Where it is drawn inside its holder. See [Placement](#placement).                        |
| `layout`      | How it arranges its own parts. See [Layout](#layout).                                     |
| `description` | What the part is for, in prose, for a reader who meets it without the file.              |

Only `name`, `tokens` and `content` are required.

## Content

`content` says what a part shows. The bare word is shorthand for `{ "type": <word> }`. Use it whenever
there is nothing else to say.

| Type           | Shows                                    | Options                                                        |
|----------------|------------------------------------------|----------------------------------------------------------------|
| `none`         | Only its own box, drawn by its tokens    | None. A divider, a track, a skeleton bar, a scrim              |
| `text`         | A run of text the consumer supplies      | `maxLines` (cut with an ellipsis), `wrap: false` (one line)    |
| `illustration` | An icon, a spinner, a drawn mark         | `name`: the component it draws, e.g. `Check`. Without one, the consumer picks it, and a preview shows a placeholder |
| `image`        | A picture the consumer supplies          | `fit`: `cover` (default, cropped) or `contain` (shown whole)   |
| `parts`        | Other parts, through its `slots`         | `slots` (required, so `parts` has no shorthand)                |

```json
"content": "none"
"content": "illustration"
"content": { "type": "text", "maxLines": 2 }
"content": { "type": "parts", "slots": { "label": { "required": true, "for": "ButtonLabel" } } }
```

## Slots

A slot is a named place in a `parts` part where another part goes. Slot names are camelCase. An
optional slot gets a switch named `show<Slot>` in code.

| Key           | Meaning                                                                                        |
|---------------|------------------------------------------------------------------------------------------------|
| `required`    | Whether the consumer must fill it.                                                             |
| `for`         | What fills it. See below.                                                                      |
| `multiple`    | `true` for a list of parts, such as the tabs of a tab list. Default `false`.                   |
| `sizing`      | `hug` (default, as big as its content) or `fill` (takes the space its siblings leave).         |
| `placement`   | Overrides the part's own placement.                                                            |
| `state`       | `linked` (default) or `own`. See [Interaction states](#interaction-states).                    |
| `config`      | The variants the held part gets from its holder. See [Configuring a held part](#configuring-a-held-part). |
| `description` | What the place is for, in prose, for whoever fills it: what belongs there, when to leave it empty. |

A slot's `description` speaks about the place, not the part in it, which has its own. It earns its
keep on open slots (`*`) and optional ones, where the reader has to decide what goes there:

```json
"extra": {
  "required": false,
  "for": "*",
  "config": "*",
  "description": "Anything the card needs under the colours, such as a delivery note or a rating."
}
```

`for` names a part of this file or another component (`"for": "Badge"`). A list allows several, and
its first entry is the default. `*` opens the slot to anything:

```json
"for": "ButtonLabel"
"for": ["CheckboxCheck", "CheckboxDash"]
"for": ["ButtonIcon", "*"]
"for": "*"
```

- `["ButtonIcon", "*"]`: `ButtonIcon` by default, and anything may replace it.
- `"*"`: no default. A preview shows a placeholder.

## Variants and states

`variants` lists the axes the consumer picks on a part, with their values. Most switch the part's
tokens, but an axis need not: `align` may only change how the part is laid out. An axis that is on
or off takes booleans:

```json
"variants": {
  "emphasis": ["high", "medium", "low"],
  "size": ["sm", "md", "lg"],
  "destructive": [false, true]
}
```

`states` lists the interaction states the part's tokens style. They are not variants: no one picks
them, they follow the user's interaction.

```json
"states": ["enabled", "hovered", "pressed", "focused", "disabled"]
```

A part that is the same whatever its holder is set to has no `variants`; one whose tokens style no
interaction has no `states`.

### Interaction states

A part shows its holder's interaction states. When a card is hovered, its title shows hover too.
A part that takes input itself sits in a slot with `"state": "own"`: each tab of a tab list, each
swatch of a colour picker, a button inside a card. Its own parts then follow it.

```json
"swatch": { "required": true, "for": "ProductCardSwatch", "multiple": true, "state": "own" }
```

The root and the parts in `own` slots are the units that take input.

### Configuring a held part

`config` on a slot says which variants the held part gets. Without it, the part gets nothing from its
holder. Each axis is set in one of three ways:

```json
"config": {
  "inherit": ["intent"],
  "emphasis": "subtle",
  "size": { "size=sm": "xs", "size=md": "sm", "size=lg": "md" }
}
```

- **Inherited** (`inherit`): the holder's value, unchanged. `"*"` passes every axis the two share.
- **Fixed**: a value, whatever the holder shows: `"emphasis": "subtle"`, `"destructive": false`.
- **By condition**: picked by the holder's variants, see [Values that follow variants](#values-that-follow-variants).
  The conditions may name any of the holder's axes (`"size": { "density=compact": "sm", … }`).
  Every holder value is covered; nothing passes unchanged, which is what `inherit` is for.

A list is shorthand for `inherit`:

```json
"config": ["intent", "emphasis"]
"config": "*"
```

`config` sets variants only. Interaction states are never configured; see
[Interaction states](#interaction-states).

## Placement

`placement` says where a part is drawn inside its holder. It can sit on the part, which applies
wherever it is placed, or on a slot, which wins.

- `flow` (default): one of the parts the holder's `layout` arranges.
- `overlay`: drawn over the holder's box, taking no space. `inline` and `block` say where it sits:
  `start`, `center`, `end` or `stretch` (default, spans the box). A bar along the bottom edge is
  `"inline": "stretch", "block": "end"`.

```json
"placement": "overlay"
"placement": { "type": "overlay", "inline": "end", "block": "start" }
```

Offsets come from tokens.

A placement can follow the variants as a whole; see [Values that follow variants](#values-that-follow-variants).

## Layout

`layout` says how a `parts` part arranges its slots. Gaps come from tokens.

```json
"layout": { "type": "flow", "direction": "row", "justify": "start", "align": "center", "wrap": true }
"layout": { "type": "grid", "columns": 6, "justify": "center", "align": "center" }
"layout": { "type": "phrase", "align": "center" }
```

- `flow`: the parts in one line along `direction` (`row` or `column`). `justify` distributes them
  along it (`start`, `center`, `end`, `space-between`, `space-around`, `space-evenly`), `align`
  places them across it (`start`, `center`, `end`, `stretch`). `wrap: true` moves them onto further
  lines when they do not fit.
- `grid`: rows of `columns` equal columns, filled in slot order. `justify` and `align` place each
  part in its cell (`start`, `center`, `end`, `stretch`, default `stretch`).
- `phrase`: the parts as one run of text, in slot order, the way a sentence is set. Text breaks
  between words and wraps; an illustration sits in the line like a letter and wraps with it.
  `align` places it against the line: `baseline` (on the baseline) or `center` (centred on the
  line).

An illustration in a `phrase` never stands alone at the start or end of a line: it keeps to the word
beside it. That is what makes a link's icon wrap with its last word instead of sitting beside the
whole label, as it would in a `flow`. How a platform keeps the two together (a word joiner, a span
that does not break) is up to the generator.

A `phrase` holds only `text`, `illustration` and `none` parts, and `sizing` and `multiple` do not
apply to its slots. Overlays still work: they are drawn over the holder's box.

Every field but `type` can follow the variants.

## Values that follow variants

Some values depend on the variants: a stacked group lays out in a column, a small card takes two
columns. Such a value is written as conditions, each giving the value it takes when it holds:

```json
"direction": { "orientation=horizontal": "row", "orientation=vertical": "column" }
```

A condition names axes and their values:

```
orientation=horizontal              one value
size=sm|lg                          any of these values
orientation=horizontal,align=start  all of these hold
size=sm|lg,align=start
destructive=true                    an axis that is on or off
```

No spaces, and nothing else: no negation, no `or` across axes, no `else`.

**Exactly one condition holds for every combination** of the axes the conditions name. There is
no default and no order, so each condition can be read on its own. Combinations are spelled out
where they matter:

```json
"justify": {
  "orientation=horizontal,align=start": "start",
  "orientation=horizontal,align=center": "center",
  "orientation=horizontal,align=end": "end",
  "orientation=vertical": "start"
}
```

The axes are the part's own `variants`; in a slot's `config`, its holder's.

| Takes conditions                                          | Fixed                                                  |
|-----------------------------------------------------------|--------------------------------------------------------|
| Every `layout` field but `type`                           | `layout.type`, `content`, `tokens`                     |
| `placement`, on the part and on the slot (as a whole)     | Slot `for`, `required`, `multiple`, `state`            |
| Slot `sizing`                                             |                                                        |
| Each axis in a slot `config`                              |                                                        |

The schema checks each condition's spelling and each value. That the axes and values exist and
that the conditions cover every combination exactly once is left to the tooling that reads the
files.

## Examples

| File                                                       | Shows                                                         |
|------------------------------------------------------------|---------------------------------------------------------------|
| [`examples/button.json`](examples/button.json)             | A small component: variants, states, an open slot, an overlay |
| [`examples/button-group.json`](examples/button-group.json) | Values that follow variants, including a combination of two axes |
| [`examples/link.json`](examples/link.json)                 | A `phrase` layout: an icon that wraps with the last word      |
| [`examples/product-card.json`](examples/product-card.json) | Every concept: images, an empty box, text limits, overlays, a grid, lists, own states, slot config |

## Repository

| Path                              | Purpose                                          |
|-----------------------------------|--------------------------------------------------|
| `schemas/<name>/v<N>.schema.json` | The schemas. One file per major version.         |
| `examples/`                       | Component files that validate against the schema |
