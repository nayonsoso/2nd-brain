# 2nd Brain — Project Rules

This folder is a 2nd brain for a backend (BE) developer.

## Writing Rules

- Use simple, easy English words. No hard or fancy words.
- Write short sentences. One idea per sentence.
- Keep notes clear and easy to read.

## Context

- Notes are about backend development topics.
- The goal is quick reference and knowledge storage, not formal writing.

## Standard Flow for Making Documents

When asked to make a document for a specific concept, always follow this flow:

1. First, tell the user how many files you are going to make and what each file is about.
2. Ask the user: are there any related concepts that are good to learn together? List them and ask if they should be included or not.
3. Wait for the user's answer before making any files.

## File Numbering

Files in a folder are numbered by learning order.
The number prefix shows the order to read them (e.g. `1-what-is-api`, `2-types-of-api`).

When you add a new file to a folder:

1. Decide where the new file fits in the learning order.
2. Renumber all files in that folder so the numbers stay in correct learning order.
3. When you rename a file, update every wikilink that points to it so no link breaks.

This rule does not apply to `personal/` and `reference/` paths.

## Review History

Every file (except those in `personal/` and `reference/` paths) must end with this section:

```
## Review History

-
```

This section must always be the last section of the file.

## Tags

Every file must have two tags on the very first line, above the title: one level tag and one status tag.

### Level Tags

| Tag           | Meaning                                                               |
| ------------- | --------------------------------------------------------------------- |
| `#aware`      | Just read it once. Good to know it exists.                            |
| `#understand` | Need a basic understanding. Know how it works.                        |
| `#essential`  | Must deeply understand. Important for any BE developer. Review often. |

### Status Tags

| Tag | Meaning |
|-----|---------|
| `#ai-draft` | First version. Written by AI. Basic. |
| `#developed` | Leveled up. More detail, sources, or examples added. |

## Wikilinks

Use Obsidian wikilinks (`[[filename]]`) to connect related notes.

## Document Format

Every file must follow this format:

- Add `---` before every `##` section (except the first one right after the tags).
- Use `==text==` to highlight key terms and important ideas.
- Write sentences on consecutive lines inside a paragraph. Do not add blank lines between sentences.
