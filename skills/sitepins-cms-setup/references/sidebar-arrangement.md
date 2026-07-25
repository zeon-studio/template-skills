# Sidebar Arrangement — `config.json` → `arrangement`

By default (`"arrangement": []`) the CMS sidebar mirrors the repo's real folder tree. The **arrangement** array lets you build a *virtual* sidebar — grouped folders, hand-picked files, and section headings — **without changing the repository structure**. It lives inside `.sitepins/config.json`.

## Item types

Three item shapes. Every item needs a unique `id` (any stable string).

### Folder group
Groups a content folder under a custom name, with optional glob filtering.
```json
{
  "id": "grp-blog",
  "type": "folder",
  "targetPath": "src/content/blog",
  "groupName": "Blog Posts",
  "include": "*.md",
  "exclude": "_*.md"
}
```
| Field | Meaning |
|-------|---------|
| `targetPath` | Real folder in the repo. |
| `groupName` | Label shown in the sidebar. |
| `include` / `exclude` | Glob patterns to show/hide files (use `""` for none). |

### File
Pins a single file into the sidebar with a custom display name.
```json
{
  "id": "file-home",
  "type": "file",
  "targetPath": "src/content/index.md",
  "groupName": "Home"
}
```
(`groupName` = the display name for the file.)

### Heading
A non-clickable section label to visually break up the sidebar.
```json
{
  "id": "head-docs",
  "type": "heading",
  "targetPath": "",
  "groupName": "Documentation"
}
```

## Example arrangement

Fragment — this is the `arrangement` key **inside** `.sitepins/config.json`, not a standalone file:

```json
"arrangement": [
  { "id": "head-1", "type": "heading", "targetPath": "", "groupName": "Content" },
  { "id": "grp-blog", "type": "folder", "targetPath": "src/content/blog", "groupName": "Blog Posts", "include": "*.md", "exclude": "" },
  { "id": "grp-authors", "type": "folder", "targetPath": "src/content/authors", "groupName": "Authors", "include": "", "exclude": "" },
  { "id": "head-2", "type": "heading", "targetPath": "", "groupName": "Pages" },
  { "id": "file-home", "type": "file", "targetPath": "src/content/index.md", "groupName": "Home" }
]
```

## Authoring guidance

- Order in the array = order in the sidebar (headings, folders, and files interleave freely).
- Arrangement is purely presentational — it never moves or renames real files, so it's always safe to change.
- Only add arrangement when the raw folder tree is awkward for editors; otherwise leave `"arrangement": []`.
- Ensure every `targetPath` (except headings) points to a path that exists in the repo.
