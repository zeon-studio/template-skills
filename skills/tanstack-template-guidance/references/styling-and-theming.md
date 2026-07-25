# Styling and Theming

These templates use **Tailwind CSS v4** together with a custom **dynamic theming engine** driven by JSON configuration.

> [!IMPORTANT]
> **Theming Verification Required:** Because this skill is used across multiple templates, the configuration and paths below are **JUST EXAMPLES**. The theming engine may be wired differently or absent. **You MUST ALWAYS verify by reading `src/config/theme.json` and inspecting `src/styles/` before making styling changes.**

## Theming engine (`theme.json`)

A Node script (commonly `scripts/themeGenerator.mjs`) reads `src/config/theme.json` and emits CSS variables into a generated stylesheet (e.g. `src/styles/generated-theme.css`). It runs automatically during `pnpm dev` and `pnpm build`.

### Changing colors

Edit `src/config/theme.json`:

```json
"colors": {
  "default": {
    "theme_color": {
      "primary": "#121212",
      "body": "#fff",
      "border": "#eaeaea"
    }
  },
  "darkmode": { }
}
```

These become CSS variables such as `--color-primary` and `--color-darkmode-primary`, consumed by Tailwind utilities.

### Changing fonts

Also in `theme.json`:

```json
"fonts": {
  "font_family": {
    "primary": "Heebo:wght@400;600",
    "primary_type": "sans-serif"
  }
}
```

## Tailwind CSS v4

Tailwind v4 is configured in CSS rather than a large `tailwind.config.js`. Look for `@import "tailwindcss"` and `@theme` blocks inside `src/styles/`. Add custom utilities and component classes there.

## Dark mode and SSR

Theme switching is a client concern, but Start renders on the server first. Reading `localStorage` during render causes a hydration mismatch and a flash of the wrong theme.

Templates normally solve this with a small inline script in the root document that sets the theme class **before** React hydrates. If you touch theming:

- Look at `src/routes/__root.tsx` for that inline script and leave it intact.
- Don't move theme resolution into a component's render path.

## Common Mistakes / What NOT to do

- **DO NOT** edit the generated theme CSS — it's overwritten on the next run. Edit `theme.json`.
- **DO NOT** hardcode hex colors in components; use the theme's Tailwind classes/variables so dark mode keeps working.
- **DO NOT** read `localStorage` during render to decide the theme — it breaks SSR hydration.
- **DO NOT** assume a `tailwind.config.js` exists; v4 configures through CSS.
