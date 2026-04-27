# Writing Instructions

This file defines authoring conventions for the cpp-field-guide. Follow these rules when writing
or editing any document in this repository.

---

## Format

- Pure Markdown. GitHub Flavored Markdown features (tables, task lists) are acceptable but not required.
- Use Mermaid for all diagrams. No ASCII art.
- No inline HTML.
- Fenced code blocks use the `cpp` language tag for C++ code.

---

## Document Structure

Every document must open with:

1. A one-sentence summary in a blockquote:
   ```
   > Brief description of what this document covers.
   ```
2. An `## Index` section with anchor links to all `##` sections, with `###` subsections indented beneath their parent.

Every `##` section must begin with a 1–2 sentence overview before any subsections or content. Use line separator between `##` sections.

For `###` headings that name a C++ type or function: use backtick code style, omit template parameters (`### \`std::optional\``, not `### \`std::optional<T>\``). In the index, use plain text without backticks — the link styling is enough.

Link text must never contain backtick code styling, even when the heading or title it references does. Use plain text inside `[…]`.

---

## Writing Style

- Be brief and concise. No filler sentences.
- Do not reproduce documentation — link to it instead.
- If a direct quote is necessary: one short excerpt + link. Nothing longer.
- Prefer a code example with a short explanation over a prose paragraph.
- Acceptable link targets:
  - **cppreference.com** for C++ documentation.
  - **godbolt.org** for runnable examples.

---

## Code Examples

- Snippets are preferred. Omit `#include` boilerplate and `main()` wrappers unless they add meaning.
- When omissions might confuse, add a `// ...` comment or a brief note.
- Use idioms that match the C++ version being documented (e.g., use `std::is_integral_v<T>` in
  C++17 content, not the C++14 `::value` form).
- Before/After comparisons use italic labels on the line before each block:

  ```
  _Before:_
  ```cpp
  // old code
  ```

  _After:_
  ```cpp
  // new code
  ```
  ```

---

## TODO Markers

- Use `TODO:` (uppercase) for in-progress content. These may remain in committed files.
- One `TODO:` per gap. Do not stack multiples on the same item.

---

## Pitfall Files (`pitfalls/`)

Each pitfall is a standalone file covering one topic. `pitfalls/README.md` is the index — it auto-renders when browsing the directory on GitHub.

Individual pitfall files open with a back-link, then a title and summary. No `## Index` needed.

```markdown
[← Pitfalls](README.md)

# Pitfall Title

Content...
```

Full runnable examples (with output blocks) are welcome in pitfall files. Use a `---` separator before each example block.

---

## Version Files (`versions/`)

Every version file follows this fixed template:

```markdown
# C++XX

> One-sentence description of what this version is about.

## Index
- [Language Features](#language-features)
  - [Feature Name](#feature-name)
- [Standard Library Additions](#standard-library-additions)
  - [Name](#name)
- [Deprecations & Removals](#deprecations--removals)

---

## Language Features

Overview sentence describing the theme of language changes in this version.

### Feature Name

...

---

## Standard Library Additions

Overview sentence describing the theme of library additions in this version.

### Name

...

---

## Deprecations & Removals

Overview sentence. List what was deprecated or removed and, briefly, why.

...
```
