# Config Setup — `.sitepins/config.json`

`.sitepins/config.json` is the **only required file**. It tells Sitepins where content, media, and config files live, how to commit, and how to arrange the sidebar.

## On-disk shape (new format)

```json
{
  "content": "src/content",
  "media": "public/images",
  "public": "public",
  "configs": ["src/config"],
  "custom-commit": false,
  "arrangement": []
}
```

| Key | Type | Meaning |
|-----|------|---------|
| `content` | string | Parent directory holding content files (Markdown/MDX/JSON/YAML/TOML). Repo-root-relative, **no leading slash**. |
| `media` | string | **Media Root** — directory where the CMS uploads/manages media files. |
| `public` | string | **Media Public** — the public base path the SSG serves media from (usually the parent of `media`). Used to build correct image URLs. |
| `configs` | string[] | Config files/folders to expose for editing (e.g. `["src/config"]`, or individual files like `["hugo.toml","src/config/menu.json"]`). Optional — use `[]` if none. |
| `custom-commit` | boolean | `true` → editor shows a commit-message dialog before each save; `false` → Sitepins auto-generates commit messages. |
| `arrangement` | array | Virtual sidebar layout. `[]` = mirror the real folder tree. See `sidebar-arrangement.md`. |

> The on-disk JSON key is `custom-commit` (hyphenated), as in real projects. The app reads it as `customCommit` internally, but **write `custom-commit` to the file**.

## Media Root vs Media Public

- **Media Root** (`media`) — the actual folder in the repo where files are stored and organized, e.g. `public/images/blog`.
- **Media Public** (`public`) — the public URL base, e.g. `public`. Ensures generated image URLs resolve on the live site.
- Rule of thumb: `public` is usually the **parent** of `media`. Selecting `public/images` as media root → `public` as media public.

## Framework detection

Detect the framework from config files at repo root, then apply its path conventions:

| Framework | Detected by (any of) | `content` | `media` | `public` | `configs` |
|-----------|----------------------|-----------|---------|----------|-----------|
| **Astro** | `astro.config.mjs` / `.js` / `.ts` | `src/content` | `public/images` | `public` | `src/config` |
| **Next.js** | `next.config.js` / `.mjs` / `.ts` | `src/content` | `public/images` | `public` | `src/config` |
| **Hugo** | `config.toml` / `config.yaml` / `config.yml` / `hugo.toml` / `hugo.yaml` / `hugo.json` | `content/<lang>` e.g. `content/english` † | `assets/images` | `static` | `config/_default`, `hugo.toml`, `data`, `i18n` |
| **Hugo (exampleSite theme)** | Hugo config **and** an `exampleSite/` folder | `exampleSite/content/<lang>` † | `exampleSite/assets/images` | `exampleSite/static` | `exampleSite/config/_default`, `exampleSite/hugo.toml`, `exampleSite/data`, `exampleSite/i18n` |

† Read the actual language dir from `config/_default/languages.toml` (`contentDir`). See **Hugo specifics** below — this choice changes how schemas must be named.

These are **suggestions, not guarantees**. Always confirm the folders actually exist in the repo before writing them. If a project deviates (e.g. Eleventy, Jekyll, Nuxt), inspect the tree and pick the real content/media directories — Sitepins is SSG-agnostic and only cares about correct paths.

**Prefer the repo's own guidance skill.** Themefisher/Gethugothemes templates ship `.agents/skills/<framework>-template-guidance/` — read its `content-management.md` / `page-configuration.md` for this template's exact content, media, and config paths rather than relying on the table.

### Hugo specifics

Hugo is **multilingual**: content sits under a language `contentDir` set in `config/_default/languages.toml` (e.g. English → `content/english/`). This choice interacts with schema naming, so decide deliberately:

- **Preferred — point `content` at the language dir** (`exampleSite/content/english`). Schema filenames then stay flat (`schema/blog.json`, `schema/authors.json`), matching what these templates already ship. Editors see one language.
- **Alternative — point `content` at the parent** (`exampleSite/content`) to expose every language in the sidebar. Then schemas **must be nested per language** (`schema/english/blog.json`), because the schema name includes the language segment. A flat `schema/blog.json` will silently never load. See the resolution table in `references/schema-authoring.md`.

Other Hugo notes:
- Media lives in `assets/images` (image module) with `static` as the public base — **not** `public/images` like Astro/Next.
- `configs` typically lists `config/_default` (or `exampleSite/config/_default`), the root `hugo.toml`, plus `data/` and `i18n/`.

Example (theme repo with `exampleSite`, language-dir approach):
```json
{
  "content": "exampleSite/content/english",
  "media": "exampleSite/assets/images",
  "public": "exampleSite/static",
  "configs": ["exampleSite/config/_default", "exampleSite/hugo.toml", "exampleSite/data", "exampleSite/i18n"],
  "custom-commit": false,
  "arrangement": []
}
```

## Authoring steps

1. Identify framework (table above). If none matches, treat as generic: find where markdown content lives and where static assets are served from.
2. Confirm each candidate folder exists (`content`, `media`, `public`).
3. For `configs`, include only real config files/folders the editor should manage (theme config, menu, site params). Skip if unsure — `[]` is safe.
4. Decide `custom-commit`: default `false` for solo/quick workflows; `true` for teams with commit conventions.
5. Set `arrangement` to `[]` unless the user wants a custom sidebar.
6. Write `.sitepins/config.json`.

## Common mistakes

- ❌ Leading slash in paths (`/src/content`). ✅ `src/content`.
- ❌ Guessing `content` without checking — a wrong content folder means Sitepins maps nothing.
- ❌ Setting `media` and `public` to the same value when the SSG strips a prefix (e.g. Astro serves `public/` at `/`).
- ❌ Writing `customCommit` to the file — the on-disk key is `custom-commit`.
