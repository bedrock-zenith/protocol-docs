# protocol-docs

Protocol files from BDS for Minecraft Bedrock.

## Why this format exists

Official Mojang docs are useful, but they are not the easiest format for parsers and code generators. This repo keeps a flat schema that is easier to process in lower-level languages.

It also keeps explicit defaults and type details that are often omitted upstream, and it records post-dump fixes when the official data is incomplete or wrong.

## Version layout

Version folders follow Mojang branch names.

- `r26_u6` is the 1.26.60 preview/stable line.
- Suffixes like `_hotfix1` mean a hotfix changed the protocol after the stable release.

## Format spec

See [FORMAT_SPEC.md](FORMAT_SPEC.md).

## Notes

This format is kept stable so downstream parsers, generators, and tools do not need to be rewritten for every upstream change.
