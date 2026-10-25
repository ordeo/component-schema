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
  "category": "actions",
  "components": [
    ...
  ]
}
```

## Files and parts

A file holds one component. `category` (required) groups it with related components by what it is for,
such as `Actions` or `Data Input`. `components` lists its parts, the root first. Every part has:

| Key           | Meaning                                                                                  |
|---------------|------------------------------------------------------------------------------------------|
| `name`        | Unique, PascalCase; by convention prefixed with its holder: `ButtonLabel` in `Button`.   |
| `tokens`      | The token paths that style the part, e.g. `component.button.label.base`.                 |
| `content`     | What the part shows. See [Content](#content).                                            |
| `variants`    | The axes the consumer picks. See [Variants and states](#variants-and-states).            |
| `states`      | The interaction states its tokens style.                                                 |
| `layout`      | How it arranges its own parts. See [Layout](#layout).                                     |
| `features`    | What kind of element it is. `interactive`: a control the user operates, such as a button. |
| `description` | What the part is for, in prose, for a reader who meets it without the file.              |

Only `name`, `tokens` and `content` are required.

## Content

`content` says what a part shows. The bare word is shorthand for `{ "type": <word> }`. Use it whenever
there is nothing else to say.

| Type           | Shows                                    | Options                                                        |
|----------------|------------------------------------------|----------------------------------------------------------------|
| `none`         | Only its own box, drawn by its tokens    | None. A divider, a track, a skeleton bar, a scrim              |
| `text`         | A run of text the consumer supplies      | `maxLines` (cut with an ellipsis), `wrap: false` (one line, whatever `maxLines` says) |
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
| `sizing`      | `hug` (default) or `fill`: in a `stack` along its direction, in a `grid` over its cell. Not in a `phrase`. |
| `placement`   | Where the part in it is drawn. See [Placement](#placement).                                    |
| `interaction` | `inherit` (default) or `own`. See [Interaction states](#interaction-states).                   |
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

`states` lists the interaction states the part's tokens style: `hovered`, `pressed`, `focused`,
`disabled`, `visited` and `readonly`. They are not variants: no one picks them, they follow the user's interaction.

```json
"states": ["hovered", "pressed", "focused", "disabled"]
```

A part that is the same whatever its holder is set to has no `variants`; one whose tokens style no
interaction has no `states`.

### Interaction states

A part shows its holder's interaction states. When a card is hovered, its title shows hover too.
A part that takes input itself sits in a slot with `"interaction": "own"`: each tab of a tab list, each
swatch of a colour picker, a button inside a card. Its own parts then follow it.

```json
"swatch": { "required": true, "for": "ProductCardSwatch", "multiple": true, "interaction": "own" }
```

The root and the parts in `own` slots are the parts that take input.

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

`placement` says where a part is drawn inside its holder. It sits on the slot only: a part never
places itself, so the same part can sit in the flow in one holder and over the box in another.

- `flow` (default): one of the parts the holder's `layout` arranges.
- `overlay`: drawn over the holder's box, taking no space. `inline` and `block` say where it sits:
  `start`, `center`, `end` or `stretch` (default, spans the box). A bar along the bottom edge is
  `"inline": "stretch", "block": "end"`. `layer` says whether it is drawn `above` (default) or
  `below` the holder's other parts. `below` keeps it over the holder's own background: the
  indicator behind the label of a picked segment.

```json
"placement": "overlay"
"placement": { "type": "overlay", "inline": "end", "block": "start" }
"placement": { "type": "overlay", "layer": "below" }
```

Offsets come from tokens.

A placement can follow the variants as a whole; see [Values that follow variants](#values-that-follow-variants).

## Layout

`layout` says how a `parts` part arranges its slots. Gaps come from tokens.

```json
"layout": { "type": "stack", "direction": "row", "justify": "start", "align": "center", "wrap": true }
"layout": { "type": "grid", "columns": 6, "justifyItems": "center", "alignItems": "center" }
"layout": { "type": "phrase", "align": "center" }
```

- `stack`: the parts in one line along `direction` (`row` or `column`). `justify` distributes them
  along it (`start`, `center`, `end`, `space-between`, `space-around`, `space-evenly`), `align`
  places them across it (`start`, `center`, `end`, `stretch`). `wrap: true` moves them onto further
  lines when they do not fit.
- `grid`: rows of `columns` equal columns, filled in slot order. `justifyItems` and `alignItems`
  place each part in its cell (`start`, `center`, `end`, `stretch`, default `stretch`).
- `phrase`: the parts as one run of text, in slot order, the way a sentence is set. A `text` part
  continues the run and wraps between words; any other part sits in the line as one piece, like a
  letter, and wraps with it. `align` places those pieces against the line: `baseline` (on the
  baseline) or `center` (centred on the line).

A `multiple` slot adds its parts to the run one after another. Overlays work as anywhere: they are
drawn over the holder's box.

`sizing` applies while a part is in the flow. In a `stack`, `fill` takes the space the siblings
leave along `direction`. In a `grid`, `fill` stretches the part over its cell, whatever
`justifyItems` and `alignItems` say; `hug` leaves it to them. An overlay is sized by its `inline` and
`block`. A `phrase` sets its parts like letters, so its slots take no `sizing`; the schema rejects
it.

Every layout also takes `block`: its box or its inline form, see [In text](#in-text). Every field but
`type` can follow the variants.

## In text

`block` on a layout says whether the part is a box of its own when it is placed in running text, or
sits in the line, such as a link in a paragraph or a badge in a sentence. Each layout has a box form
and an inline form:

| Layout   | `block: true` (default)                | `block: false`                                             | Example |
|----------|----------------------------------------|------------------------------------------------------------|---------|
| `stack`  | A box (`flex`)                         | One piece in the line, moves with it (`inline-flex`)       | Badge   |
| `grid`   | A box (`grid`)                         | One piece in the line, moves with it (`inline-grid`)       |         |
| `phrase` | A box with a run of text, a paragraph  | Continues the text around it and wraps with it (`inline`)  | Link    |

```json
"layout": { "type": "phrase", "align": "center", "block": false }
```

A box placed in text breaks it and stands on lines of its own. In a box, every part is a box. A part
without a layout is in the line by its nature: text continues the run, anything else is one piece.

It can follow the variants. A link that can also stand on its own, as a line of its own under a
teaser, is a box only then:

```json
"variants": { "standalone": [false, true] },
"layout": { "type": "phrase", "align": "center", "block": { "standalone=false": false, "standalone=true": true } }
```

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

The axes are the `variants` of the part the value is written in. A slot belongs to its holder, so
its `placement`, `sizing` and `config` follow the holder's.

| Takes conditions                                          | Fixed                                                  |
|-----------------------------------------------------------|--------------------------------------------------------|
| Every `layout` field but `type`                           | `layout.type`, the `content` type, `tokens`            |
| Slot `placement` (as a whole)                             | Slot `for`, `required`, `multiple`, `interaction`      |
| Slot `sizing`                                             |                                                        |
| Text `maxLines` and `wrap`                                |                                                        |
| Each axis in a slot `config`                              |                                                        |

The schema checks each condition's spelling and each value. That the axes and values exist and
that the conditions cover every combination exactly once is left to the tooling that reads the
files.

## Examples

| File                                                       | Shows                                                         |
|------------------------------------------------------------|---------------------------------------------------------------|
| [`examples/button.json`](examples/button.json)             | A small component: variants, states, an open slot, an overlay |
| [`examples/button-group.json`](examples/button-group.json) | Values that follow variants, including a combination of two axes |
| [`examples/link.json`](examples/link.json)                 | A `phrase` layout with `block: false`: a link that wraps with its paragraph |
| [`examples/segmented-control.json`](examples/segmented-control.json) | An overlay with `layer: below`: an indicator drawn behind the label |
| [`examples/product-card.json`](examples/product-card.json) | Every concept: images, an empty box, text limits, overlays, a grid, lists, own states, slot config |

## Repository

| Path                              | Purpose                                          |
|-----------------------------------|--------------------------------------------------|
| `schemas/<name>/v<N>.schema.json` | The schemas. One file per major version.         |
| `examples/`                       | Component files that validate against the schema |
