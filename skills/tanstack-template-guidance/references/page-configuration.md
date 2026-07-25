# Page and Site Configuration

This skill explains how to configure global settings, navigation, social links, and SEO metadata in this TanStack Start template.

## Global Configuration Files

Configuration typically lives as JSON files in `src/config/`.

> [!IMPORTANT]
> **Configuration Verification Required:** Because this skill is used across multiple templates, the files and schemas below are **JUST EXAMPLES**. They may be missing, renamed, or contain entirely different keys. **You MUST ALWAYS list `src/config/` and read the actual file contents before reading or modifying any configuration value.**

## Example Configuration Files (Verify Before Use)

### `src/config/config.json`
Master configuration for the site.
- **`site`** — `title`, `base_url`, `favicon`, `logo` paths and dimensions
- **`settings`** — feature flags such as `search`, `sticky_header`, `theme_switcher`, `default_theme`, pagination limits
- **`params`** — global parameters like `contact_form_action`, `copyright`
- **`metadata`** — global SEO defaults (`meta_author`, `meta_image`, `meta_description`)

### `src/config/menu.json`
Header and footer navigation.
- **`main`** — header menu array (`name`, `url`), often supporting nested children
- **`footer`** — footer links

### `src/config/social.json`
Social profile links consumed by a `<Social />` component. Empty values usually hide the icon.

### `src/config/theme.json`
Colors and fonts — see `references/styling-and-theming.md`.

## SEO Metadata

TanStack Router manages document head data per route. Most templates set titles and meta tags through a route's `head` option, which merges down the route tree — the root route supplies defaults and child routes override them.

Resolution order is typically:

1. **Per-page** — frontmatter of the markdown file (`title`, `meta_title`, `description`, `image`), passed into the route's head config
2. **Global fallback** — `config.json` → `metadata`

> Verify how *this* template does it: read `src/routes/__root.tsx` and a leaf route before adding metadata. Some templates use a `<SeoMeta />` component instead. Using both produces duplicate tags.

## Navigation and type-safe links

Menu entries from `menu.json` are rendered through `<Link>`. Because TanStack Router type-checks `to` against the generated route tree, a menu URL with no matching route becomes a **type error** rather than a silent 404 — useful, but it means adding a menu entry may require adding the route first.

For fully dynamic URLs coming from JSON, templates often widen the type or use an `href` escape hatch. Follow whatever pattern the template already uses.

## Common Mistakes / What NOT to do

- **DO NOT** delete config keys without checking usage — prefer an `enable: false` toggle where one exists.
- **DO NOT** hardcode nav links into header/footer components; keep them in `menu.json`.
- **DO NOT** use a relative `base_url` — it must be fully qualified (e.g. `https://yourdomain.com`).
- **DO NOT** set the same meta tag in both a route `head` and a component; pick one mechanism.
