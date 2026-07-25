# Snippet Authoring — `.sitepins/snippet/*.json`

Snippets are **reusable blocks of code** — Hugo shortcodes, MDX/JSX components, or HTML — that editors insert into content from the editor toolbar. Each snippet is one small JSON file in `.sitepins/snippet/`, version-controlled with the repo.

## Snippet file shape

```json
{
  "label": "Button",
  "schema": [],
  "code": "<Button label=\"\" href=\"\" />\n"
}
```

| Key | Type | Meaning |
|-----|------|---------|
| `label` | string | Friendly name shown in the editor's snippet list (e.g. "Button", "YouTube Embed"). |
| `schema` | string[] | Schema **scoping** — see the matching rule below. `[]` = available everywhere. `["blog"]` = only while editing a file inside a `blog/` folder. |
| `code` | string | The exact code inserted at the cursor. Use `\n` for newlines; blank attribute values (`href=""`) give the editor placeholders to fill. |

**Filename:** kebab/lowercase after the label, e.g. `post-aside.json`, `button.json`.

## How `schema` scoping actually matches

Snippet scoping does **not** use the schema-file path. It compares each `schema` entry against the **basename of the folder containing the file being edited**, case-insensitively:

- Editing `src/content/blog/post-1.md` → context is `blog` → snippets with `"schema": ["blog"]` (or `[]`) appear.
- Editing `exampleSite/content/english/blog/post-1.md` → context is still `blog`. Language nesting is irrelevant, so **the same `["blog"]` value works for Hugo** even though the *schema file* may live at `schema/english/blog.json`.
- `"schema": []` (or the key omitted) → always available.
- With no file open, only unrestricted snippets (`[]`) are shown.

So scope values are **folder names**, which usually — but not always — coincide with schema names.

## Examples

Global inline component:
```json
{ "label": "A", "schema": [], "code": "<A href=\"\" rel=\"\" label=\"\" />" }
```

Blog-only block components:
```json
{ "label": "Mockup", "schema": ["blog"], "code": "<Mockup src=\"/blog/image.webp\" alt=\"alt text\" />\n" }
```
```json
{ "label": "Post Aside", "schema": ["blog"], "code": "<PostAside toc=\"h2\" promotion />\n" }
```

Hugo shortcode:
```json
{ "label": "YouTube", "schema": [], "code": "{{< youtube id=\"\" >}}\n" }
```

## How to author snippets

1. Find the components/shortcodes actually used in the project's content (grep existing `.md`/`.mdx` for custom tags like `<Button`, `<Notice`, `{{< ... >}}`), or the components exported for MDX use.
2. Create one snippet per reusable element with sensible empty attribute placeholders.
3. Scope with `schema` only when a snippet is content-type-specific; otherwise use `[]`.
4. Write each to `.sitepins/snippet/<label>.json`.

## Notes

- Snippets are optional — a project is fully functional without any.
- Block-level snippets inserted mid-paragraph are auto-converted to an inline form by the editor; author the natural block form.
- Keep `label`s descriptive of what the block does, not cryptic codes.
