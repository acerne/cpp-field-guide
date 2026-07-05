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

Every document must open with a one-sentence summary in a blockquote:

```
> Brief description of what this document covers.
```

Every `##` section must begin with a 1–2 sentence overview before any subsections or content. Use line separator between `##` sections in articles and version files. Do not use separators in category index files (`README.md`).

For `###` headings that name a C++ type or function: use backtick code style, omit template parameters (`### \`std::optional\``, not `### \`std::optional<T>\``).

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
- Never put commands that are not meant to run together in the same fenced code block. This includes
  per-platform install commands or "pick one" alternatives. Each platform gets its own subsection
  with its own code block.
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

## Article Files (`pitfalls/`, `idioms/`)

Each article is a standalone file covering one topic. The directory `README.md` is the category index and auto-renders on GitHub.

Articles open with a back-link, then a title and blockquote summary:

```markdown
[← Category Name](README.md)

# Article Title

> One-sentence summary.

Content...
```

Pitfall articles welcome full runnable examples (with output blocks). Use a `---` separator before each example block.

---

## Guide Files (`guides/`)

Each guide is a standalone file. Articles open with a back-link, title, and blockquote summary — identical to idiom articles. Use `---` separators between `##` sections.

**Feature guides** explain how to use a specific language or library feature. Structure sections freely by usage pattern.

**Design decision guides** compare multiple approaches and end with a `## Prefer` section stating the rule in 1–2 plain sentences. The opening blockquote must already state the recommendation.

**Version tags**: Feature guides include a version tag on the line immediately after the H1 title:

```
_C++17_
```

If the article covers features from multiple C++ versions, move the tag to the relevant `##` section heading instead of the top. Design decision guides with no version dependency omit the tag entirely.

**cppreference links**: Link to cppreference at the end of the first `##` section that introduces the type or function. One link per article; if the article covers multiple distinct types, one link per type at first introduction.

```markdown
[← Guides](README.md)

# Topic

> One-sentence recommendation stated directly.

---

## Option A Name

Brief explanation.

```cpp
// example
```

---

## Option B Name

Brief explanation.

```cpp
// example
```

---

## Prefer

State the rule plainly. One to two sentences, no hedging.
```

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
