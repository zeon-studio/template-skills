# Data Loading

Data in TanStack Start comes from **route loaders** and **server functions**. Understanding which runs where is the main thing to get right.

## Route loaders

Every route can declare a `loader`. It runs before the component renders and its result is available via `Route.useLoaderData()` with full type inference.

```tsx
export const Route = createFileRoute('/blog/$slug')({
  loader: ({ params }) => getPost(params.slug),
  component: PostPage,
})

function PostPage() {
  const post = Route.useLoaderData()
  return <article>{post.title}</article>
}
```

**Where does a loader run?** On the initial SSR request it runs on the server. On subsequent client-side navigations it runs in the browser. So a loader must be isomorphic — it **cannot** directly touch the database, secrets, or `fs`. Put that work in a server function.

## Server functions

`createServerFn` marks work as server-only. It gives validation, a serializable boundary, and same-origin RPC semantics — the client calls it like a normal async function while the implementation never ships to the browser.

```ts
import { createServerFn } from '@tanstack/react-start'

export const getPost = createServerFn({ method: 'GET' })
  .validator((slug: string) => slug)
  .handler(async ({ data: slug }) => {
    // Safe: runs only on the server
    return readPostFromDisk(slug)
  })
```

Use a server function whenever the work involves the database, auth, environment variables, secrets, or the filesystem — including reading markdown off disk.

Server functions can be called from loaders, components, and handlers alike.

## Choosing between them

| Need | Use |
|---|---|
| Data for a route, available before render | Route `loader` |
| Database / auth / env / `fs` access | `createServerFn` (call it from the loader) |
| Mutation triggered by user action | `createServerFn`, called from the component |
| Static content baked at build time | Build-time collections — see `references/content-management.md` |

## Rendering modes

Start can render the full document, stream UI progressively, or opt individual routes into SPA / selective-SSR modes — all while preserving client-side Router interactivity. Check the template's `router.tsx` and route definitions before assuming everything is fully SSR'd.

## Common Mistakes / What NOT to do

- **DO NOT** import `fs`, database clients, or secret env vars directly in a loader or component — the loader may run in the browser. Wrap them in `createServerFn`.
- **DO NOT** fetch in a `useEffect` for data the route needs; use the loader so it's ready before render and works under SSR.
- **DO NOT** return non-serializable values (class instances, functions) from a server function — they must cross a serialization boundary.
- **DO NOT** assume loader data is fresh forever; check the route's caching/staleness options if content looks stale.
