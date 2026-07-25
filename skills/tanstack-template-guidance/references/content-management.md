# Content Management

This skill explains how markdown content is managed in a TanStack Start template.

## The key difference from other frameworks

**TanStack Start has no built-in content layer.** Astro has Content Collections, Hugo pairs content with layouts — TanStack Start has neither. Markdown handling is whatever the template wired up.

> [!IMPORTANT]
> **Content Verification Required:** You MUST determine which approach this template uses *before* creating or editing content. Check, in order:
> 1. Is there a `content-collections.ts` at the repo root? → build-time collections
> 2. Does `package.json` list `@content-collections/core`, `gray-matter`, `unified`, or an MDX plugin?
> 3. Grep `src/` for `gray-matter`, `import.meta.glob`, or `createServerFn` reading files
>
> The content directory is **whatever that config points at** — do not assume `src/content/`.

## Approach 1: Build-time collections (most common for blogs)

The `content-collections` package processes markdown at build time, giving type-safe imports and fast runtime reads. Content lives in a directory named in `content-collections.ts` (e.g. `./src/content/blog` or `./src/blog`).

A collection typically declares its directory, a glob for files, and a transform that runs `gray-matter` to split frontmatter from the body. Routes then import the generated collection (e.g. `allPosts`) and filter or sort it.

**Trade-off:** content is baked in at build time, so publishing a change requires a rebuild.

## Approach 2: Runtime loading via server functions

`createServerFn` fetches markdown at request time — from disk or a remote source such as GitHub's raw content API — and parses it with `gray-matter`. Useful when content must update without a redeploy.

**Trade-off:** runtime overhead, and it needs real error handling for missing/malformed files.

Both approaches usually funnel the parsed markdown through a shared `renderMarkdown()` helper built on the `unified` ecosystem.

## Example Content Structure (Verify Before Use)

```text
src/content/
├── blog/{post-name}.md
├── authors/{author-name}.md
├── pages/{page-name}.md
└── sections/{section-name}.md
```

## Frontmatter

Every markdown file starts with YAML frontmatter. Fields depend entirely on what the template's loader and components read, so **mirror an existing file in the same folder** rather than guessing.

Commonly:

- `title` — String, the main title
- `date` — ISO date string (e.g. `2024-01-15T05:00:00Z`)
- `description` — Short summary for listings and SEO
- `image` — Path to the cover image (starts with `/images/`)
- `draft` — Boolean; typically excluded from production builds
- `categories` / `tags` — Arrays
- `author` — Should match an entry in `authors/`

If the template uses `content-collections`, its transform/schema is the **authoritative** field list — read it.

## File Naming Conventions

- Kebab-case filenames: `my-new-post.md`
- The filename usually becomes the URL slug, consumed by a `$slug.tsx` route
- Check how the route derives the slug before renaming files — some templates use a frontmatter `slug` field instead

## Image Handling

- Images go in `public/images/`
- Reference them with an absolute path from the web root: `/images/my-image.jpg`
- `public/` is served at `/`, so `public/images/foo.png` → `/images/foo.png`

## Common Mistakes / What NOT to do

- **DO NOT** assume `src/content/` exists — read the content config first.
- **DO NOT** put markdown in `src/routes/`. Files there become URLs; content and routes are separate concerns.
- **DO NOT** use relative image paths like `../../public/images/x.jpg`. Always `/images/...`.
- **DO NOT** add a frontmatter key the loader doesn't read and expect it to appear — wire it into the collection schema/transform and the component.
- **DO NOT** forget that build-time collections need a rebuild (or dev-server restart) to pick up new files.
