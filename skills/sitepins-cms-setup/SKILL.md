---
name: sitepins-cms-setup
description: Use this skill to configure a project (repository) for the Sitepins git-based headless CMS by generating the `.sitepins/` folder — `config.json`, content `schema/*.json`, reusable `snippet/*.json`, and sidebar `arrangement`. Use it whenever the user asks to "set up Sitepins", "add Sitepins to this repo", "create Sitepins schemas/snippets/config", "configure the CMS", "generate .sitepins", or wants any Sitepins settings/schema/snippet file authored for a static-site project (Astro, Next.js, TanStack Start, Hugo, Eleventy, Jekyll, etc.).
---

# Sitepins CMS Setup

Sitepins is a **git-based headless CMS**. It never hosts or renders the site — it reads existing content files (Markdown/MDX/JSON/YAML/TOML) straight from the git repository and overlays a visual editor. All CMS behavior for a project lives in **one folder committed to the repo**:

```
.sitepins/
  config.json        # required — folder mapping + commit + sidebar arrangement
  schema/*.json      # optional — content templates per content folder
  snippet/*.json     # optional — reusable shortcode/JSX/HTML blocks
```

Normally a user clicks through the Sitepins web UI to produce these files. **This skill's job is to author them directly** by inspecting the repo, so the project is CMS-ready without the UI.

## Golden rules

1. **Everything is committed JSON inside `.sitepins/`.** Never invent a different location. Constants are fixed: schema folder = `.sitepins/schema`, snippet folder = `.sitepins/snippet`, config = `.sitepins/config.json`.
2. **Ready before you configure, per framework.** Content must live in the content folder as frontmatter files — Sitepins cannot manage content hardcoded in components/config/`src/data`. Astro/Next.js/TanStack Start/Hugo differ (paths, loaders, validation); defer to the repo's `<framework>-template-guidance` skill and audit/convert first (`references/cms-readiness.md`).
3. **Inspect before you author.** Read the real repo — detect the framework, list content folders, open a representative content file per folder — and derive schemas from actual frontmatter. Do not hardcode fields from memory.
4. **Paths in `config.json` are repo-root-relative, no leading slash.**
5. **Use the real runtime field-type vocabulary** (`string`, `number`, `boolean`, `Date`, `media`, `gallery`, `Array`, `object`, plus auto-detected `color`) — not loose names like `text`/`textarea`/`datetime`/`select`. Long-form text (`description`, `content`) is **`string`**, not `textarea`. See `references/schema-authoring.md`.
6. **Schemas only for collections, not singletons.** A schema templates *new* files, so it belongs on folders that grow (`blog/`, `authors/`, `pages/`) — not on one-off pages (`homepage/`, `about/`, `contact/`), which editors open and edit in place. Ask: would anyone click "add new file" here?
7. **One schema JSON per collection, named by the resolution rule.** Schema filename = the content folder path **minus the `config.content` root** (`src/content/blog` → `blog.json`; `exampleSite/content` + `…/english/blog` → `english/blog.json`). A misnamed schema silently never loads. Subfolders inherit the parent. See `references/schema-authoring.md`.

## Workflow

1. **Detect the framework** — `astro.config.*` → astro, `next.config.*` → nextjs, `config.toml`/`hugo.*` → hugo (`exampleSite/` present → `hugo_examplesite`), a bundler config (`vite.config.*`/`rsbuild.config.*`/`app.config.*`) **plus** routing evidence (`routeTree.gen.ts` or `routes/__root.tsx`) → tanstack.
2. **Load the template's guidance skill** — Astro, Next.js, TanStack Start, and Hugo store and load content differently, so the repo's own `.agents/skills/<framework>-template-guidance/` is the authoritative content model. **If it isn't installed, install it first:** `npx skills add zeon-studio/template-skills --skill <framework>-template-guidance`.
3. **Check CMS readiness** — Sitepins can only manage content that lives in the content folder as frontmatter files. If the template hardcodes content in components/config/`src/data`, it is **not ready**: convert it to a content-folder-driven structure before configuring anything. → `references/cms-readiness.md`
4. **Map folders → `config.json`** using framework conventions. → `references/config-setup.md`
5. **Author schemas — for collection folders only.** Skip singletons like `homepage/`, `about/`, `contact/` (a lone `-index.md`/`_index.md` is the tell). For each remaining collection, open one existing file, read its frontmatter, and write the schema under the name given by the resolution rule (content folder path minus `config.content`). → `references/schema-authoring.md`
6. **Author snippets** (optional) — for shortcodes/components used in content. → `references/snippet-authoring.md`
7. **Author sidebar arrangement** (optional) — virtual folders/files/headings inside `config.json`. → `references/sidebar-arrangement.md`
8. **Verify before reporting:**
   - Every path in `config.json` exists in the repo, is repo-root-relative, and has no leading slash.
   - Every schema filename equals `<content folder> − <config.content>` (compute it, don't assume) — this is the #1 silent failure.
   - No schema was written for a singleton folder (one `-index.md`/`_index.md` and nothing else).
   - Every schema's `file` points at a content file that actually exists; `fileType`/`fmType` match it.
   - All JSON parses, and field `type` values come from the allowed vocabulary.
9. **Report** the files created and how each was derived.

## Routing guide

Read the reference file that matches the task before writing any JSON:

- **Is the template CMS-ready? Auditing/converting hardcoded content into the content folder** → `references/cms-readiness.md`
- **`config.json`, framework detection, content/media/public/configs paths, commit mode** → `references/config-setup.md`
- **Content schemas, all field types, nested/array/media/dropdown/reference fields, inheritance** → `references/schema-authoring.md`
- **Reusable snippets (shortcodes/JSX/HTML), schema scoping** → `references/snippet-authoring.md`
- **Sidebar arrangement (virtual folders, files, headings, glob include/exclude)** → `references/sidebar-arrangement.md`

## Reference: a complete `.sitepins/` for an Astro project

`config.json`
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

`schema/blog.json` (derived from a real `src/content/blog/*.md`)
```json
{
  "file": "src/content/blog/example-post.md",
  "name": "blog",
  "fileType": "md",
  "fmType": "yaml",
  "template": [
    { "name": "title", "label": "Title", "type": "string", "value": "", "isRequired": true },
    { "name": "date", "label": "Date", "type": "Date", "value": "", "alwaysUseCurrentDate": false },
    { "name": "image", "label": "Image", "type": "media", "value": "" },
    { "name": "draft", "label": "Draft", "type": "boolean", "value": false },
    { "name": "categories", "label": "Categories", "type": "Array", "value": [] }
  ]
}
```

`snippet/button.json`
```json
{ "label": "Button", "schema": [], "code": "<Button label=\"\" href=\"\" />\n" }
```

Always verify against the actual files in the target repo before committing.
