# Adding New Pages

Routes are **file-based**: a file in `src/routes/` becomes a URL. The TanStack Router plugin watches this directory and regenerates `routeTree.gen.ts` automatically during dev and build.

> [!IMPORTANT]
> **Verify before use.** The routes directory is usually `src/routes/`, but a template may configure another location. Check the router plugin options in `vite.config.ts` (or `rsbuild.config.ts` / `app.config.ts`) and list the directory before adding files.

## Naming conventions

| File | URL | Purpose |
|---|---|---|
| `__root.tsx` | — | Root layout wrapping **every** route. Holds the `<html>` shell, nav, footer. |
| `index.tsx` | `/` | Exact match for the parent path |
| `about.tsx` | `/about` | Static route |
| `blog/index.tsx` | `/blog` | Index of a nested directory |
| `blog/$slug.tsx` | `/blog/:slug` | Dynamic param — `$` prefix |
| `blog/$.tsx` | `/blog/*` | Splat / catch-all |
| `_layout.tsx` | — | **Pathless** layout — wraps children, adds no URL segment |
| `_layout/dashboard.tsx` | `/dashboard` | Child of the pathless layout |

Two structural styles, freely mixable:

- **Directory routing** — nested folders mirror the URL (`blog/$slug.tsx`)
- **Flat routing** — dots denote nesting (`blog.$slug.tsx`)

## Adding a static page

Create `src/routes/about.tsx`:

```tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/about')({
  component: AboutPage,
})

function AboutPage() {
  return <h1>About</h1>
}
```

Save it — the plugin regenerates the route tree and the page is live at `/about`. **Never** hand-edit `routeTree.gen.ts`.

## Adding a dynamic page

Create `src/routes/blog/$slug.tsx`. Access the param via `Route.useParams()`, and load data in the route's `loader`:

```tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/blog/$slug')({
  loader: ({ params }) => getPost(params.slug),
  component: PostPage,
})

function PostPage() {
  const post = Route.useLoaderData()
  return <article>{post.title}</article>
}
```

See `references/data-loading.md` for loaders and server functions.

## Linking between pages

Use `Link` — never a bare `<a>` for internal navigation, or you lose client-side routing and type safety:

```tsx
import { Link } from '@tanstack/react-router'

<Link to="/blog/$slug" params={{ slug: 'my-post' }}>Read post</Link>
```

Links are **type-checked against the generated route tree**: an invalid `to` or a missing param is a TypeScript error, not a runtime 404.

## Common Mistakes / What NOT to do

- **DO NOT** edit `routeTree.gen.ts` — it is regenerated and your changes vanish.
- **DO NOT** use `<a href="/about">` for internal links; use `<Link to="/about">`.
- **DO NOT** forget the `$` prefix for params — `blog/slug.tsx` creates the literal URL `/blog/slug`.
- **DO NOT** assume a missing route means a build error; if the dev server was not running when you added the file, restart it so the tree regenerates.
