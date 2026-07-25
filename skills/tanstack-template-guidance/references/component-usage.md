# Component Usage

This skill explains how the UI component architecture is structured in this TanStack Start template and how to use the existing components.

## Component Architecture

Components typically live in `src/components/`, with larger page sections sometimes split into a `partials/` or `sections/` subfolder. Route files in `src/routes/` compose these components — keep routes thin and components reusable.

> [!IMPORTANT]
> **Component Verification Required:** Because this skill is used across multiple templates, the components listed below are **JUST EXAMPLES**. Components may be missing, renamed, or structured differently depending on the specific template. **You MUST ALWAYS verify component existence and read their actual implementation by listing `src/components/` before attempting to use or modify them.**

## Example Components (Verify Before Use)

### `<BlogCard />`
- **Purpose**: Displays a single blog post preview.
- **Props**: Receives a `data`/`post` object containing frontmatter and parsed content.
- **Usage**: Mapped over the array returned by a route loader in `src/routes/blog/index.tsx`.

### `<SeoMeta />`
- **Purpose**: Sets document title and meta tags.
- **Note**: TanStack Router can manage document head data through a route's `head` option, so some templates set metadata there instead of via a component. **Check the route definitions before adding a component-based approach** — mixing both leads to duplicate tags.

### `<ThemeSwitcher />`
- **Purpose**: Toggles light/dark mode, usually persisting to `localStorage` and toggling a class on the root element.
- **Note**: Must be client-safe. Guard against reading `window`/`localStorage` during SSR, or hydration will mismatch.

### `<Logo />`
- **Purpose**: Renders the site logo, pulling source URLs from `config.json`.

## Composing components in routes

Route components receive their data from the route's `loader`, then pass it down as props:

```tsx
function BlogIndex() {
  const posts = Route.useLoaderData()
  return posts.map((post) => <BlogCard key={post.slug} data={post} />)
}
```

Keep UI components stateless where possible; do the fetching in the loader (see `references/data-loading.md`).

## SSR considerations

TanStack Start renders on the server by default, so components must survive a server pass:

- Guard browser-only APIs (`window`, `document`, `localStorage`) behind an effect or a mounted check.
- Don't generate random values or timestamps during render — server and client output will differ and hydration will warn.

## Common Mistakes / What NOT to do

- **DO NOT** fetch data inside atomic components. Load it in the route's loader and pass it down.
- **DO NOT** hardcode the site name, logo, or nav links into components — read them from `src/config/`.
- **DO NOT** touch `window` or `localStorage` during the initial render without a guard; it breaks SSR.
- **DO NOT** assume a component exists because it's listed here. Verify it in the codebase first.
