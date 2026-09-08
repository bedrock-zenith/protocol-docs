# Format specification

## Common rule

Every object has:

- `name`: `string`
- `bind_type`: `string`
- `is_bind_type`: `boolean`

Every file of type has:

- `hash`: `string` // ps: we will generate hashes for whole namespace as well soon

## Type: `namespace`

`__protocol__.json`

- `name`: `string`
- `bind_type`: `"namespace"`
- `is_bind_type`: `boolean` (`false`)
- `enums`: `array of objects`

`__protocol_canonical__.json`

- `name`: `string`
- `bind_type`: `"namespace"`
- `is_bind_type`: `boolean` (`false`)
- `enums`: `array of objects` ordered by usage

## Type: `enum`

`*.enum.json`

- `name`: `string`
- `hash`: `string`
- `bind_type`: `"enum"`
- `is_bind_type`: `boolean` (`true`)
- `enum_kind`: `string`
- `backing_integer`: `string or null`
- `enum`: `object with string keys and number values`
- `exhaustive`: `string or null`
- `enum_encoding`: `string or null`
- `data_encoding`: `string, object, or null`

Notes:

- This is the type definition.
- The same enum may be referenced from multiple fields.
- A field may encode the enum differently than the enum definition itself.
- Multiple keys can map to the same value. Those are enum aliases.
- Example: `{ "Idle": 0, "None": 0 }`

## Type: `struct`

`*.struct.json`

- `name`: `string`
- `hash`: `string`
- `bind_type`: `"struct"`
- `is_bind_type`: `boolean` (`true`)
- `fields`: `array of objects`

Field:

- `name`: `string`
- `is_constant`: `boolean`
- `type`: `object`

Type reference example:

- `{ "name": "Actor Link Type", "bind_type": "enum", "is_bind_type": true }`

Type definition example:

- `{ "name": "string", "is_bind_type": false, "length_encoding": { ... } }`

Notes:

- Structs are flat.
- A field stores a reference to a bind type when the type is defined elsewhere.
- A field may also contain encoding metadata for that field only.

## Type: `alias`

`*.alias.json`

- `name`: `string`
- `hash`: `string`
- `bind_type`: `"alias"`
- `is_bind_type`: `boolean` (`true`)
- `element_type`: `object`

Notes:

- Alias is a named wrapper around another type.
- It is still a bind type because it has its own name and file.

## Primitive and composed types

These are not bind types:

- `string`
- `boolean`
- `float`
- `integer`
- `array`
- `optional`
- `hash`

Array:

- `name`: `"array"`
- `is_bind_type`: `boolean` (`false`)
- `element_type`: `object`
- `length_encoding`: `object or null`
- `data_encoding`: `string or null`

Integer / numeric:

- `name`: `string`
- `is_bind_type`: `boolean` (`false`)
- `interpretation`: `string`
- `reinterpret`: `string or null`
- `data_encoding`: `string or null`

## Encoding and definition

Definition fields:

- `name`: `string`
- `bind_type`: `string`
- `is_bind_type`: `boolean`
- `hash`: `string`
- `element_type`: `object or null`
- `enum`: `object with string keys and number values or null`
- `fields`: `array of objects or null`
- `interpretation`: `string or null`
- `backing_integer`: `string or null`
- `enum_kind`: `string or null`

Encoding-only fields:

- `data_encoding`: `string, object, or null`
- `length_encoding`: `object or null`
- `enum_encoding`: `string or null`
- `exhaustive`: `string or null`
- `reinterpret`: `string or null`

Important split:

- type definition describes what the type is
- field encoding describes how this field stores that type in this struct

Example:

- enum bind type: `Actor Link Type`
- field type: `{ "name": "Actor Link Type", "bind_type": "enum", "is_bind_type": true, "data_encoding": { ... } }`

This means:

- the enum may be defined once globally
- the same enum can be encoded differently in different fields
- the field-local `data_encoding` is not the type definition itself
- it is the encoding used at this location

Dumper rule:

- `StringEncodingInformation` adds `length_encoding`
- `ArrayEncodingInformation` adds `length_encoding`
- `NumberEncodingInformation` adds `reinterpret` and `data_encoding`
- `EnumEncodingInformation` adds `enum_encoding`, `exhaustive`, and `data_encoding`
- `BooleanEncodingInformation` adds `data_encoding`

## Naming

- Names use Pascal Spaced Case.
- Example: `Actor Link Type`, `Add Player Packet Payload`
- This makes conversion to kebab-case, snake_case, camelCase, and others simple.

## Layout

- Each bind type is in its own file.
- Files are under version folders such as `r26_u6`.
- Folder names follow Mojang branch names.
- `r26_u6` means the 1.26.60 preview/stable line.
- `_hotfix1` and similar suffixes mean a hotfix changed the protocol after the stable release.

## Why this format exists

- easier to parse from lower-level languages
- explicit defaults instead of hidden assumptions
- stable layout across upstream changes
- flat type references instead of nested definitions
- enum deduplication by content
- post-dump fixes when official docs are incomplete or incorrect

## Canonical ordering

`__protocol_canonical__.json` is the same protocol data, but ordered by dependency.

- if a type is used by others, it is placed earlier
- this helps in languages where type definition order matters, such as JS classes and C/C++
- it is mainly a stable emit order, not a different schema
