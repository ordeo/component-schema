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
| `variants`    | The axes its tokens branch on. See [Variants and states](#variants-and-states).          |
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
| `text`         | A run of text the consumer supplies      | `maxLines` (cut with an ellipsis), `wrap: false` (one line)    |
| `illustration` | An icon, a spinner, a drawn mark         | `name`: the component it draws, e.g. `Check`. Without one, the consumer picks it, and a preview shows a placeholder |
| `image`        | A picture the consumer supplies          | `fit`: `cover` (default, cropped) or `contain` (shown whole)   |
| `parts`        | Other parts, through its `slots`         | `slots` (required, so `parts` has no shorthand)                |

```json
"content": "illustration"
"content": { "type": "text", "maxLines": 2 }
"content": { "type": "parts", "slots": { "label": { "required": true, "for": "ButtonLabel" } } }
```

## Slots

A slot is a named place in a `parts` part where another part goes. Slot names are camelCase. An
optional slot gets a switch named `show<Slot>` in code.

| Key         | Meaning                                                                                        |
|-------------|------------------------------------------------------------------------------------------------|
| `required`  | Whether the consumer must fill it.                                                             |
| `for`       | What fills it. See below.                                                                      |
| `multiple`  | `true` for a list of parts, such as the tabs of a tab list. Default `false`.                   |
| `sizing`    | `hug` (default, as big as its content) or `fill` (takes the space its siblings leave).          |
| `placement` | Overrides the part's own placement.                                                            |
| `state`     | `linked` (default) or `own`. See [Interaction states](#interaction-states).                    |
| `config`    | The variants the held part gets from its holder. See [Configuring a held part](#configuring-a-held-part). |

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

`variants` lists the axes a part's tokens branch on, with their values. An axis that is on or off
takes booleans:

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

Both list only what the part's own tokens do, so a part whose look never changes has neither.

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
  "size": { "from": "size", "map": { "sm": "xs", "md": "sm", "lg": "md" } }
}
```

- **Inherited** (`inherit`): the holder's value, unchanged. `"*"` passes every axis the two share.
- **Fixed**: a value, whatever the holder shows: `"emphasis": "subtle"`, `"destructive": false`.
- **Mapped** (`from`, `map`): one of the holder's axes, translated. `from` may name another axis
  (`"size": { "from": "density", … }`). A value not listed in `map` passes unchanged.

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

## Layout

`layout` says how a `parts` part arranges its slots. Gaps come from tokens.

```json
"layout": { "type": "flow", "direction": "row", "justify": "start", "align": "center", "wrap": true }
"layout": { "type": "grid", "columns": 6, "justify": "center", "align": "center" }
```

- `flow`: the parts in one line along `direction` (`row` or `column`). `justify` distributes them
  along it (`start`, `center`, `end`, `space-between`, `space-around`, `space-evenly`), `align`
  places them across it (`start`, `center`, `end`, `stretch`). `wrap: true` moves them onto further
  lines when they do not fit.
- `grid`: rows of `columns` equal columns, filled in slot order. `justify` and `align` place each
  part in its cell (`start`, `center`, `end`, `stretch`, default `stretch`).

## Examples

| File                                                       | Shows                                                         |
|------------------------------------------------------------|---------------------------------------------------------------|
| [`examples/button.json`](examples/button.json)             | A small component: variants, states, an open slot, an overlay |
| [`examples/product-card.json`](examples/product-card.json) | Every concept: images, text limits, overlays, a grid, lists, own states, slot config |

## Repository

| Path                              | Purpose                                          |
|-----------------------------------|--------------------------------------------------|
| `schemas/<name>/v<N>.schema.json` | The schemas. One file per major version.         |
| `examples/`                       | Component files that validate against the schema |
