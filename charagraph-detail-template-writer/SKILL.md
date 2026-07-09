---
name: charagraph-detail-template-writer
description: create valid charagraph detail template schemas as copy-pastable json or yaml when the user asks for a detail template, character detail schema, detail set structure, or fields for character information such as glasses, hairstyle, facial hair, education, health, outfits, personality, family, habits, or other reusable character profile sections. use this to turn plain-language requirements into a clean charagraph template with sections, supported field types, tables, media galleries, conditional visiblewhen rules, and optional auto-tagging. output concise json/yaml code ready to paste into charagraph.
---

# CharaGraph Detail Template Writer

## Core behavior

Create CharaGraph Detail Template schemas from the user's plain-language description.

Prefer direct generation over long discussion. Ask at most one focused clarification only when the requested template is too ambiguous to produce a useful schema.

Use the user's requested format:
- Output JSON when the user asks for JSON.
- Output YAML when the user asks for YAML.
- Output both only when explicitly requested.
- If no format is specified, default to YAML because it is easier to read and edit.

Always output a copy-pastable fenced code block. Keep explanation outside the code block brief, or omit explanation entirely when the user only wants the template.

Do not output escaped JSON strings. Do not wrap the schema inside API payloads unless the user asks for that.

## CharaGraph template shape

Generate templates with this root structure:

```yaml
title: Template Name
description: Short user-facing description.
version: "1.0.0"
sections:
  - key: section_key
    title: Section Title
    description: Optional section description.
    fields:
      - key: field_key
        label: Field Label
        type: text
```

Use stable `snake_case` keys. Do not use spaces or hyphens in keys.

Keep section keys unique. Keep field keys unique across the template, especially when fields are referenced by `visibleWhen` or auto-tagging.

Set `version` to `"1.0.0"` for a new template unless the user asks otherwise.

## Supported field types

Use only CharaGraph-supported field types:

```text
text
textarea
number
boolean
select
radio
multi_select
rating
color
url
date
date_range
character_reference
table
media_gallery
```

Choose the simplest field type that fits the data.

Use:
- `text` for short free-form values.
- `textarea` for notes, descriptions, routines, and longer prose.
- `number` for numeric measurements or counts.
- `boolean` for true/false facts.
- `select` for one choice from many.
- `radio` for a small one-choice set where visible options are useful.
- `multi_select` for multiple choices.
- `rating` for scored subjective values.
- `color` for color values.
- `url` for links.
- `date` for one date.
- `date_range` for start/end periods.
- `character_reference` for linking another character.
- `table` for repeating structured history.
- `media_gallery` for images/videos/reference media attached to the character.

## Field conventions

Use `label`, `type`, and usually `description` for clarity. Use `required: true` sparingly.

Use type-compatible defaults:

```yaml
# text / textarea / select / radio / date / color / url
defaultValue: ""

# number / rating
defaultValue: null

# boolean
defaultValue: false

# multi_select
defaultValue: []

# date_range
defaultValue:
  start: ""
  end: ""

# table / media_gallery
defaultValue: []
```

For `select`, `radio`, and `multi_select`, include `options` as an array of strings:

```yaml
options:
  - Short
  - Medium
  - Long
```

For `rating`, include practical bounds:

```yaml
type: rating
min: 1
max: 5
step: 1
```

For `character_reference`, include `allowMultiple` when relevant:

```yaml
type: character_reference
allowMultiple: false
```

## Table fields

Use a `table` when the user describes repeating history, multiple records, logs, appointments, changes over time, or power/history data.

Table example:

```yaml
- key: hairstyle_history
  label: Hairstyle History
  type: table
  description: Record past hairstyles and changes over time.
  allowAddRows: true
  allowDeleteRows: true
  allowReorderRows: true
  defaultValue: []
  columns:
    - key: period
      label: Period
      type: text
    - key: style
      label: Style
      type: text
    - key: notes
      label: Notes
      type: textarea
```

Use table column types conservatively: `text`, `textarea`, `number`, `boolean`, `select`, and `date` are safest.

Do not set table `defaultValue` to an empty string. Use `[]`.

## Media gallery fields

Use `media_gallery` for images, videos, references, and visual examples.

```yaml
- key: hairstyle_media
  label: Hairstyle Media
  type: media_gallery
  description: Images or videos showing the character's hairstyle.
  accept: image
  allowMultiple: true
  maxItems: 12
  defaultValue: []
```

Allowed `accept` values:

```text
image
video
any
```

Prefer `image` unless the user specifically mentions video.

## Conditional visibility

Use `visibleWhen` when the user describes fields that should appear only for certain answers.

Supported operators:

```text
equals
not_equals
includes
not_includes
is_true
is_false
is_empty
is_not_empty
```

Simple condition:

```yaml
visibleWhen:
  field: facial_hair_status
  operator: equals
  value: Beard
```

Grouped condition:

```yaml
visibleWhen:
  all:
    - field: has_facial_hair
      operator: is_true
    - field: facial_hair_status
      operator: equals
      value: Beard
```

Rules:
- Reference existing field keys exactly.
- Do not make a field depend on itself.
- Use `equals`/`not_equals` for select and radio.
- Use `includes`/`not_includes` for multi_select.
- Use `is_true`/`is_false` for boolean.
- Use values that exactly match option strings.
- Hidden values are preserved by CharaGraph, so do not create duplicate fallback fields unless the user asks.

## Auto-tagging

Add `tagging` only when the user asks for auto-tags, categories, tag generation, or searchable classification. Do not over-tag every field by default.

For select/radio/multi_select fields, use concise internal tags and friendly visible labels:

```yaml
tagging:
  enabled: true
  tagFormat: "hair.length: {value}"
  labelFormat: "{value} hair"
  optionLabels:
    Short: short hair
    Medium: medium hair
    Long: long hair
```

For boolean fields, prefer clear true/false labels when auto-tagging is requested:

```yaml
tagging:
  enabled: true
  tagFormat: "wears.glasses: {value}"
  labelFormat: "{value}"
  optionLabels:
    "true": wears glasses
    "false": does not wear glasses
```

Keep internal tags stable and lowercase where practical. Visible labels should be natural text.

## Output style

When generating a full template:
- Use a descriptive title.
- Use 3 to 7 sections unless the user asks for a very small or very large template.
- Group related fields logically.
- Put general/current state fields first.
- Put preferences/details next.
- Put history tables later.
- Put media fields near the end.
- Keep labels user-friendly.
- Keep descriptions short.
- Include only fields that are useful for character information.

When the user gives a rough idea, infer sensible fields. When the user gives an exact field list, follow it closely and do not add too many extras.

When the user asks for both JSON and YAML, output two separate fenced code blocks, JSON first only if the user requested JSON first.

## JSON rules

When outputting JSON:
- Use valid JSON only.
- Use double quotes for all keys and strings.
- Do not include comments.
- Do not include trailing commas.
- Use `null`, `true`, and `false` correctly.

## YAML rules

When outputting YAML:
- Use two-space indentation.
- Quote version strings, dates, and ambiguous values when helpful.
- Quote strings containing `:` if needed.
- Do not include comments unless the user explicitly asks for annotated YAML.

## Quality checklist before answering

Verify:
- root has `title`, `description`, `version`, and `sections`.
- each section has `key`, `title`, and `fields`.
- each field has `key`, `label`, and `type`.
- all keys are snake_case.
- all field types are supported.
- all select/radio/multi_select fields have options.
- all table fields have columns and `defaultValue: []`.
- all media_gallery fields have `accept`, `allowMultiple`, and `defaultValue: []`.
- all visibleWhen references point to existing field keys.
- default values match field types.
- the output is copy-pastable JSON/YAML.

## Example interaction

User:

```text
Create a YAML template for facial hair. It should ask if the character has facial hair. If yes, show beard/mustache details and media.
```

Assistant output:

```yaml
title: Facial Hair Details
description: Tracks facial hair status, style, care, history, and reference media.
version: "1.0.0"
sections:
  - key: overview
    title: Overview
    fields:
      - key: has_facial_hair
        label: Has Facial Hair
        type: boolean
        defaultValue: false
      - key: facial_hair_status
        label: Facial Hair Status
        type: radio
        options:
          - Clean Shaven
          - Beard
          - Mustache
          - Beard and Mustache
        defaultValue: Clean Shaven
  - key: beard_details
    title: Beard Details
    visibleWhen:
      field: facial_hair_status
      operator: includes
      value: Beard
    fields:
      - key: beard_style
        label: Beard Style
        type: text
        defaultValue: ""
      - key: beard_notes
        label: Beard Notes
        type: textarea
        defaultValue: ""
  - key: media
    title: Media
    fields:
      - key: facial_hair_media
        label: Facial Hair Media
        type: media_gallery
        accept: image
        allowMultiple: true
        maxItems: 12
        defaultValue: []
```

Prefer correcting obvious operator mistakes in final output. For the example above, if `facial_hair_status` is a radio field with value `Beard and Mustache`, use a condition structure that matches the intended options, such as `any` with `equals` conditions, rather than invalid `includes` on a radio field.
