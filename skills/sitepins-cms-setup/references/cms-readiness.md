# CMS Readiness — is the template git-CMS ready?

Sitepins manages **content files** (frontmatter Markdown/MDX/JSON/YAML/TOML) in the repo. It cannot manage content hardcoded in components, config modules, or `data`/`.ts` files. So **before** generating `.sitepins/`, audit the template and, if needed, **convert it to a content-folder-driven structure**.

> Run this step FIRST. A `.sitepins/config.json` pointing at a content folder that doesn't hold the site's real content produces an empty/useless CMS.

## Core principle (framework-agnostic)

**All editable content MUST live in the content folder as frontmatter files** — homepage sections, static pages, blog posts, authors, reusable sections (CTA/testimonials/features/FAQ), and any copy an editor would change. If it's inlined in components/config/`data`, it's **not ready**.

**But the *how* differs per framework.** Astro, Next.js, and Hugo store, load, and validate content differently — do not apply one structure to all three.

## Use the project's own template-guidance skill (authoritative)

Themefisher/Gethugothemes templates ship an installed guidance skill inside the repo at:

```
.agents/skills/<framework>-template-guidance/references/
  content-management.md      ← content model, folders, frontmatter, gotchas
  project-architecture.md    ← how content is loaded/rendered
  page-configuration.md      ← config files (site config, menu, params)
```

`<framework>` is one of `astro`, `nextjs`, `hugo`. **Read that repo's `content-management.md` and `project-architecture.md` first** — they are the source of truth for *this* template's content layout, loader wiring, and required frontmatter. This Sitepins skill and those guidance skills come from the same `template-skills` repo, so they agree by design.

### Install the guidance skill if it's missing

Do not assume it's present — check first, install if not:

1. **Detect the framework** (`astro.config.*` → astro, `next.config.*` → nextjs, `config.toml`/`hugo.*` → hugo).
2. **Check for the skill** at `.agents/skills/<framework>-template-guidance/`.
3. **If missing, install it** from the `template-skills` repo, then read it:
   ```bash
   npx skills add zeon-studio/template-skills --skill astro-template-guidance   # or nextjs-/hugo-template-guidance
   ```
   Pick the one matching the detected framework. (This is the same command used to install this Sitepins skill.)
4. **Only fall back** to reading raw sources if install isn't possible: Astro → `src/content.config.ts`; Next.js → the content parser (`src/lib/contentParser.ts` or similar); Hugo → `config/_default/languages.toml` for `contentDir`.

## Readiness audit

1. **Load the framework guidance** (above) so you know where content is *supposed* to live for this template.
2. **Locate the content folder** (per-framework table below). Missing → not ready.
3. **Grep for hardcoded content** outside it:
   - Literal strings / JSX text in `src/components`, `src/pages`, `src/app`, `src/layouts`, Hugo `layouts/`.
   - Content arrays/objects in `src/data/**`, `src/config/**`, `*.ts`/`*.js` exports (`export const features = [...]`).
   - Section text passed as props instead of read from content.
4. **Check loader wiring** (per-framework table below).
5. **Verdict:** real content already in the content folder and loaders read it → ✅ ready, skip to schema generation. Otherwise → ❌ convert first.

## Per-framework readiness

| | **Astro** (`astroplate`) | **Next.js** (`nextplate`) | **Hugo** (`hugoplate`) |
|---|---|---|---|
| Content dir | `src/content/` | `src/content/` | language `contentDir`, e.g. `content/english/` (or `exampleSite/content/english/` for theme repos) |
| Media dir | `public/images/` | `public/images/` | `assets/images/` |
| Media public base | `public` | `public` | `static` (or `exampleSite/static`) |
| Loader | Astro Content Collections; **Zod schema in `src/content.config.ts`** (build fails on mismatch) | Custom parsers (`gray-matter`), e.g. `src/lib/contentParser.ts` — dir is hardcoded | Hugo content/layout pairing; templates read frontmatter directly |
| Index/landing file | `-index.md` / `index.md` per template convention | same | `_index.md` = branch bundle (section landing); `page/index.md` = leaf bundle |
| Config files | `src/config/*.json` | `src/config/*.json` | `config/_default/*.toml`, `hugo.toml`, `data/`, `i18n/` |
| Key gotchas | Frontmatter must satisfy `content.config.ts` Zod. Don't guess keys — read it. | Keys must match parser exactly (`categories` not `category`). Content only parsed from `src/content/`. | **Multilingual**: content is under a language dir — read `languages.toml` before assuming paths. Section files (`sections/*.md`) need `build.render: "never"`. Author `author` must match an author `title`. Images via image module partial, not raw `<img>`. |

> **Astro & Next.js** share `src/content/` + `public/images`, but Astro adds strict Zod validation — schema-derived `.sitepins` fields must match `content.config.ts`. **Hugo** is different across the board: language-nested content, `assets/images`, `static` public base, and `_index.md` bundles.

## Target content structure (Astro / Next.js)

```
src/content/
├── homepage/-index.md     # homepage sections (nested frontmatter)
├── sections/{name}.md     # reusable sections (CTA, testimonials, ...)
├── pages/{name}.md        # static pages (privacy, terms, ...)
├── blog/{post}.md
├── authors/{author}.md
├── about/index.md
└── contact/index.md
```

For **Hugo**, mirror the existing language tree instead (`content/english/blog/`, `content/english/authors/`, `content/english/sections/`, `content/english/about/_index.md`), and follow that repo's `content-management.md`.

## Frontmatter patterns (verify against the template's existing files / guidance skill)

**Homepage** (nested sections):
```yaml
---
banner:
  title: "Main headline"
  content: "Supporting text"
  image: "/images/banner.png"
  button: { enable: true, label: "Get Started", link: "/start" }
features:
  - title: "Feature"
    image: "/images/feature.png"
    content: "Description"
    bulletpoints: ["Point 1", "Point 2"]
---
```

**Reusable section** (`sections/{name}.md`) — add `build: { render: "never" }` on Hugo:
```yaml
---
enable: true
title: "Section Title"
description: "Section description"
image: "/images/section.png"
button: { enable: true, label: "Call to Action", link: "/action" }
---
```

**Blog post**:
```yaml
---
title: "Post Title"
meta_title: "SEO Title"
description: "Meta description"
date: 2024-01-15T05:00:00Z
image: "/images/post.png"
categories: ["Category"]
author: "Author Name"
tags: ["tag1"]
draft: false
---
Post body in Markdown...
```

**Author**:
```yaml
---
title: "Author Name"
email: "author@example.com"
image: "/images/avatar.png"
description: "Brief bio"
social:
  - { name: github, icon: FaGithub, link: https://github.com/username }
---
Bio in Markdown...
```

**Static page**:
```yaml
---
title: "Page Title"
meta_title: "SEO Title"
description: "Meta description"
image: "/images/page.png"
draft: false
---
Page body in Markdown...
```

## Conversion procedure (not-ready → ready)

For each piece of hardcoded content:

1. **Create the content file** in the framework's correct folder, moving copy into frontmatter (structured data) + Markdown body (prose).
2. **Wire the loader to read it** — follow the template's `project-architecture.md` / `content-management.md`:
   - **Astro**: add/extend the collection in `src/content.config.ts` (Zod), then replace inline text with `getEntry()`/`getCollection()`.
   - **Next.js**: read via the content helper (`getSinglePage()`/`getListPage()` / `contentParser.ts`) and render from parsed frontmatter/body.
   - **Hugo**: place the file under the language `contentDir`; render via the section's layout/partial (image module for images). Add `build.render: "never"` for reusable section files.
3. **Remove the hardcoded copy** from the component/config/data file (keep markup, drop the literal).
4. **Verify the page renders identically** (dev server / build). On Astro, the build also validates the Zod schema.

### Conversion rules

MUST:
- ✅ Store all editable content in the framework's content folder as frontmatter files.
- ✅ YAML frontmatter for structured data, Markdown body for prose.
- ✅ Consistent field naming (`title`, `description`, `image`, `button`, `enable`, ...).
- ✅ `enable: true/false` for toggleable sections.
- ✅ Images via the framework's convention (`/images/...`; Hugo image module partial).

MUST NOT:
- ❌ Leave editable content hardcoded in components/`layouts`.
- ❌ Keep content in `src/data/`, `src/config/`, or arbitrary `.ts`/`.js` exports.
- ❌ Apply one framework's paths to another (see the table).
- ❌ Change layout/markup or visual output — this is a data move, not a redesign.

## After conversion

1. Point `config.json` `content` at the framework's real content folder (`references/config-setup.md`).
2. Generate schemas from the **actual** content files, matching any Zod/parser requirements (`references/schema-authoring.md`).
3. Continue with snippets / arrangement as needed.
