# Schema Authoring — `.sitepins/schema/*.json`

A schema is a **content template**: it defines the frontmatter fields, types, and defaults used when an editor creates a **new** file in a folder. Sitepins pre-fills the new file with the template's fields. One JSON file per collection, stored in `.sitepins/schema/`.

Because schemas only drive *file creation*, they belong on folders that grow — not on one-off pages. See "Which folders actually need a schema" before writing any.

## Schema file shape

```json
{
  "file": "src/content/blog/example-post.md",
  "name": "blog",
  "fileType": "md",
  "fmType": "yaml",
  "template": []
}
```

`template` holds the ordered array of field objects described below.

| Key | Meaning |
|-----|---------|
| `file` | Path to a **real existing content file** used as the source/example. Repo-relative. |
| `name` | Human label for the schema (usually the content folder, e.g. `blog`). **Cosmetic only** — resolution is by file path, not this field. Keep it consistent with the filename anyway. |
| `fileType` | `md` or `mdx`. |
| `fmType` | Frontmatter format: `yaml` (`---` delimiters) or `toml` (`+++` delimiters). Detect from the source file. |
| `template` | Ordered array of field definitions (below). |

**Filename is what binds a schema to a folder** — get it wrong and the schema silently never loads. See "Schema resolution" below before naming the file.

## Field object

```json
{
  "name": "title",
  "label": "Title",
  "type": "string",
  "value": "",
  "defaultValue": "",
  "description": "",
  "isRequired": false,
  "isIgnored": false
}
```

| Property | Type | Notes |
|----------|------|-------|
| `name` | string | Frontmatter key (e.g. `title`, `meta_title`). Must be **unique within the schema**. |
| `label` | string | Display name in the editor. |
| `type` | string | One of the field types below. |
| `value` | any | Seed value written into new files. Match the type (see per-type defaults). |
| `defaultValue` | any | Optional default used when generating a new file. |
| `description` | string | Optional help text shown in the editor. |
| `isRequired` | boolean | Editor blocks save if empty. |
| `isIgnored` | boolean | Field parsed but hidden/left untouched by the editor. |
| `subType` | string | **`Array` fields only** — `"string"` or `"object"`. See below. |
| `isDropdown` + `reference*` | — | Turns a `string` / `Array`-of-string field into a dropdown. See "Dropdown / reference fields". |

## Field types (runtime vocabulary — use exactly these)

| `type` | Editor control | `value` default | Extra properties |
|--------|----------------|-----------------|------------------|
| `string` | text input | `""` | Use for **all** text, including long-form `description`/`content` |
| `number` | numeric input | `0` | — |
| `boolean` | toggle | `false` | — |
| `Date` | date picker | `""` | `alwaysUseCurrentDate` (bool) — if `true`, stamps today's date on save |
| `media` | single media picker | `""` | value is a path under the media folder |
| `gallery` | multi media picker | `[]` | array of media paths |
| `Array` | repeatable list | `[]` | `subType` (`"string"` \| `"object"`); plus `fields` when `subType` is `object` |
| `object` | nested group | — | requires `fields` — nested field array |
| `color` | color picker | `""` | for hex/color-string values |

The first eight are exactly what the schema editor's type menu offers. `color` is the one extra type — it is **auto-detected** when a value looks like a color (e.g. `#ff0000`), not chosen from the menu.

> **Capitalization is significant** — `Date` and `Array` are capitalised; `string`/`number`/`boolean`/`object`/`media`/`gallery`/`color` are lowercase.
>
> **There is no `textarea` type.** Long-form fields such as `description` and `content` are plain `string`. Nor is there `text`, `datetime`, `select`, or `list` — do not invent aliases.

### Nested objects and arrays of objects

Use `fields` to nest. An `object` groups fields; an `Array` with `subType: "object"` is a repeatable group.

```json
{
  "name": "social",
  "label": "Social",
  "type": "Array",
  "subType": "object",
  "value": [],
  "fields": [
    { "name": "icon", "label": "Icon", "type": "media", "value": "" },
    { "name": "link", "label": "Link", "type": "string", "value": "" }
  ]
}
```

### `subType` (Array fields)

An `Array` field **must** declare what it holds. This drives everything else about it:

| `subType` | Meaning |
|-----------|---------|
| `"string"` | List of scalars. Can be turned into a **multi-select dropdown**. |
| `"object"` | Repeatable group — requires a nested `fields` array. |

`Array` defaults to `subType: "string"` when created. Omitting it on a list of scalars is the usual reason a multi-select won't offer dropdown mode.

### Dropdown / reference fields

A dropdown is a **plain field plus `isDropdown: true`** — it is not its own `type`. Only two shapes support it:

- `type: "string"` → single-select dropdown
- `type: "Array"` + `subType: "string"` → multi-select dropdown

Options come from one of three sources, set by `referenceType`.

**1. Static options** — a fixed list you supply:
```json
{
  "name": "status",
  "label": "Status",
  "type": "string",
  "value": "",
  "isDropdown": true,
  "referenceType": "static",
  "options": ["draft", "review", "published"]
}
```

**2. Files from a folder** — one option per file (e.g. pick an author):
```json
{
  "name": "author",
  "label": "Author",
  "type": "string",
  "value": "",
  "isDropdown": true,
  "referenceType": "folder",
  "referencePath": "src/content/authors",
  "referenceInclude": "*.md",
  "referenceExclude": "_*.md",
  "referenceField": "title"
}
```

**3. Data from a file** — options read from a key inside a JSON/YAML/TOML file. Combine with `Array` for multi-select:
```json
{
  "name": "categories",
  "label": "Categories",
  "type": "Array",
  "subType": "string",
  "value": [],
  "isDropdown": true,
  "referenceType": "file",
  "referencePath": "src/config/taxonomy.json",
  "referenceField": "categories"
}
```

| Property | Meaning |
|----------|---------|
| `isDropdown` | Enables dropdown mode. Required. |
| `referenceType` | `static` \| `folder` \| `file` — where options come from. |
| `options` | `string[]` of choices. **`static` only.** |
| `referencePath` | The folder (`folder`) or the data file (`file`) to read. Not used by `static`. |
| `referenceInclude` / `referenceExclude` | Glob filters. **`folder` only.** |
| `referenceField` | Dual purpose — for `folder`, the frontmatter key shown as each option's label (omit/empty ⇒ use the filename; `slug` is also available). For `file`, the key to extract from the data file (omit/empty ⇒ use the whole file/array). |

Only emit the keys the chosen `referenceType` actually uses; stray `options` or glob keys on a folder/file source are ignored but make the schema misleading.

## Which folders actually need a schema

**A schema is a template for creating _new_ files. Folders that never gain new files don't need one.** Authoring schemas for them is noise — it clutters Schema Management and implies a "create new" workflow that doesn't exist.

Apply this test to each content folder:

| Folder looks like | Kind | Schema? |
|---|---|---|
| Many sibling content files that grow over time — `blog/`, `authors/`, `products/`, `pages/`, `docs/` | **Collection** | ✅ Yes |
| A single landing file only (`-index.md` / `_index.md` / `index.md`) — `homepage/`, `about/`, `contact/` | **Singleton** | ❌ No |
| A fixed set of files tied to specific layouts — `sections/` (CTA, testimonial) | **Fixed set** | ⚠️ Only if editors genuinely add new ones |

The quick heuristic: **would an editor ever click "add new file" here?** If no, skip the schema — they'll just open the existing file and edit it in place.

Worked example (a typical Astro/Next template):

```
src/content/
  homepage/-index.md          → singleton, no schema
  about/-index.md             → singleton, no schema
  contact/-index.md           → singleton, no schema
  sections/call-to-action.md  → fixed set, usually no schema
  blog/post-1.md …            → collection  ✅ schema/blog.json
  authors/john-doe.md …       → collection  ✅ schema/authors.json
  pages/privacy-policy.md …   → collection  ✅ schema/pages.json
```

Section landing files (`-index.md` inside a collection) are themselves singletons — they don't change the fact that the folder around them is a collection.

## How to author a schema (do this per collection folder)

1. List the content folders under `config.content`, then **keep only the collections** per the test above (e.g. `blog/`, `authors/`, `pages/`).
2. For each, **open one representative existing file** and read its frontmatter.
3. Map each frontmatter key to a field:
   - any text, short or long (including `description` / `content`) → `string`
   - number → `number`; boolean → `boolean`
   - date-looking value → `Date`
   - image/media path → `media`; array of media → `gallery`
   - array of scalars → `Array` + `subType: "string"`; array of objects → `Array` + `subType: "object"` + `fields`
   - nested map → `object` + `fields`
   - hex color → `color`
4. Preserve the field **order** from the source file.
5. Set `isRequired: true` on essentials (`title`, `date`).
6. Set `fileType`/`fmType` from the source file; set `file` to that source path; set `name` to the folder.
7. Write the schema using the filename from the resolution rule below.

## Schema resolution (naming rule — get this right)

The schema filename is derived from the content folder **with the `config.content` root stripped off**:

```
schemaName = <folder path>  minus  <config.content>       (leading/trailing slashes trimmed)
schema file = .sitepins/schema/<schemaName>.json
```

Sitepins then tries three lookups, in order:

1. **Exact match** — `.sitepins/schema/<schemaName>.json`. Note `schemaName` **keeps its slashes**, so nested folders map to *nested schema files*.
2. **Root collection** — the first path segment of `schemaName`, i.e. `.sitepins/schema/<first-segment>.json`.
3. **Ancestor walk (inheritance)** — walks up parent folders looking for a schema.

Worked examples:

| `config.content` | Content folder | schemaName | 1. Exact | 2. Root collection |
|---|---|---|---|---|
| `src/content` | `src/content/blog` | `blog` | `schema/blog.json` | `schema/blog.json` |
| `src/content` | `src/content/blog/2024` | `blog/2024` | `schema/blog/2024.json` | `schema/blog.json` ✅ |
| `exampleSite/content` | `exampleSite/content/english/blog` | `english/blog` | `schema/english/blog.json` | `schema/english.json` ⚠️ |
| `exampleSite/content/english` | `exampleSite/content/english/blog` | `blog` | `schema/blog.json` ✅ | `schema/blog.json` |

**Inheritance** is really lookup #2 + #3: `blog/2024` has no schema of its own, so it falls back to `blog.json`. Create one schema per content section and nested year/category folders reuse it automatically.

### ⚠️ Hugo multilingual pitfall

Hugo keeps content under a **language directory** (`content/english/…`). If `config.content` points at the *parent* (`exampleSite/content`), then `schemaName` for the blog folder is `english/blog` — so a flat `.sitepins/schema/blog.json` **is never found** (lookup #2 resolves to `english.json`, not `blog.json`).

Two valid fixes — pick one and stay consistent:

- **Preferred:** point `config.content` at the language dir (`exampleSite/content/english`) → schemas stay flat (`schema/blog.json`, `schema/authors.json`). Works with the schema files these templates already ship.
- **Alternative:** keep `config.content` at the parent and nest schemas to match (`schema/english/blog.json`).

Always compute `schemaName` from the real `config.content` you wrote before choosing filenames.

## Full example — products schema

```json
{
  "file": "content/products/example.md",
  "name": "products",
  "fileType": "md",
  "fmType": "yaml",
  "template": [
    { "name": "title", "label": "Title", "type": "string", "value": "", "isRequired": true },
    { "name": "date", "label": "Date", "type": "Date", "value": "", "alwaysUseCurrentDate": false },
    { "name": "description", "label": "Description", "type": "string", "value": "" },
    { "name": "price", "label": "Price", "type": "number", "value": 0 },
    { "name": "image", "label": "Image", "type": "media", "value": "" },
    { "name": "categories", "label": "Categories", "type": "Array", "value": [] },
    { "name": "draft", "label": "Draft", "type": "boolean", "value": false }
  ]
}
```
