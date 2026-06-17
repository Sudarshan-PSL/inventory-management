---
name: saas-redesign
description: Redesign the Vue 3 frontend into a modern SaaS-style interface with a left vertical navigation sidebar (replacing the top nav bar), a consistent spacing scale, and a polished professional look. Use when asked to modernize the UI, add a sidebar layout, improve visual polish, or make the app feel like a SaaS product.
---

# SaaS UI Redesign

Transform the inventory-management frontend from its current **top-nav** layout into a modern **SaaS app shell**: a fixed vertical sidebar on the left for navigation, a clean content area on the right, a single source-of-truth design-token system, and consistent spacing throughout.

This skill is **visual/structural only**. It must not change data flow, API contracts, routes, i18n keys, or any behavior. Every screen that worked before must work after.

## Non-negotiable rules

1. **Delegate all `.vue` work to the `vue-expert` subagent.** Per the project's mandatory rule, any time you create or significantly modify a `.vue` file you MUST use the `vue-expert` agent. This skill's job is to hand `vue-expert` a precise spec (the tokens, the sidebar structure, the layout shell, the polish checklist below) and then verify the result. Do not hand-edit `.vue` files yourself.
2. **Backend is off-limits.** Do not touch `server/`, `server/data/*.json`, or `client/src/api.js` contracts.
3. **Preserve all functionality:** every `router-link` and route, `ProfileMenu`, `LanguageSwitcher`, `FilterBar`, the profile/tasks modals, and all `t('...')` i18n keys must remain wired exactly as they are. You are moving and restyling these elements, not removing them.
4. **No emojis in the UI** (business app). Use clean SVG icons or text.
5. **No new heavy dependencies.** Icons should be inline SVG or a lightweight set — do not add a UI framework (Tailwind, Vuetify, etc.) unless the user explicitly asks.

## Step 1 — Establish the design-token system

The current styles in `client/src/App.vue` hardcode colors and ad-hoc spacing (`1.25rem`, `0.875rem`, `0.625rem`…). Replace that with CSS custom properties defined once on `:root` in the global `<style>` of `App.vue`. Everything downstream references these tokens.

```css
:root {
  /* Brand / accent */
  --color-accent:        #2563eb;
  --color-accent-hover:  #1d4ed8;
  --color-accent-soft:   #eff6ff;

  /* Sidebar (dark slate shell — the SaaS signature) */
  --sidebar-bg:          #0f172a;
  --sidebar-fg:          #94a3b8;   /* idle link text   */
  --sidebar-fg-active:   #ffffff;   /* active link text */
  --sidebar-active-bg:   #1e293b;   /* active link pill */
  --sidebar-border:      #1e293b;

  /* Content surfaces */
  --bg:           #f8fafc;
  --surface:      #ffffff;
  --border:       #e2e8f0;
  --border-strong:#cbd5e1;

  /* Text */
  --text:         #0f172a;
  --text-muted:   #64748b;
  --text-subtle:  #94a3b8;

  /* Status */
  --success: #059669;  --success-soft: #d1fae5;
  --warning: #ea580c;  --warning-soft: #fed7aa;
  --danger:  #dc2626;  --danger-soft:  #fecaca;
  --info:    #2563eb;  --info-soft:    #dbeafe;

  /* Spacing scale — 4px base, USE ONLY THESE */
  --space-1: 0.25rem;  /*  4px */
  --space-2: 0.5rem;   /*  8px */
  --space-3: 0.75rem;  /* 12px */
  --space-4: 1rem;     /* 16px */
  --space-5: 1.5rem;   /* 24px */
  --space-6: 2rem;     /* 32px */
  --space-8: 3rem;     /* 48px */

  /* Radius / elevation */
  --radius-sm: 6px;
  --radius:    10px;
  --radius-lg: 14px;
  --shadow-sm: 0 1px 2px rgba(15,23,42,.06);
  --shadow:    0 4px 12px rgba(15,23,42,.06);
  --shadow-lg: 0 10px 24px rgba(15,23,42,.10);

  /* Layout */
  --sidebar-w:         260px;
  --sidebar-w-collapsed: 72px;
  --content-max:       1400px;
}
```

**Consistent spacing is enforced by the scale.** When restyling, replace every magic-number margin/padding/gap with a `--space-*` token. No value outside the scale unless there's a documented reason. Cards use `--space-5` padding, grids use `--space-5` gaps, stacked sections use `--space-5` vertical rhythm.

## Step 2 — Build the sidebar shell (the core change)

Extract navigation into a new component `client/src/components/Sidebar.vue` and rebuild `App.vue` as a two-column shell.

**New `App.vue` layout** (from a vertical `column` stack to a `row` shell):

```
.app  (display:flex; flex-direction:row; min-height:100vh)
├── <Sidebar />            fixed width var(--sidebar-w), full height, dark
└── .app-main  (flex:1; min-width:0; display:flex; flex-direction:column)
    ├── .topbar            slim bar: page context + LanguageSwitcher + ProfileMenu
    ├── <FilterBar />      stays, but sits inside .app-main
    └── <main>  <router-view />   padded container, max-width var(--content-max)
```

The top-of-page controls that don't belong in the sidebar (`LanguageSwitcher`, `ProfileMenu`) move into a slim **`.topbar`** at the top of `.app-main` — OR into the sidebar footer. Default: keep `LanguageSwitcher` + `ProfileMenu` in a slim topbar so the sidebar stays focused on navigation; put a compact user summary in the sidebar footer.

**`Sidebar.vue` structure:**

```vue
<template>
  <aside class="sidebar" :class="{ collapsed }">
    <!-- Brand -->
    <div class="sidebar-brand">
      <span class="brand-mark">{{ /* logo glyph */ }}</span>
      <div class="brand-text" v-show="!collapsed">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
      </div>
    </div>

    <!-- Primary nav (preserve EVERY existing route + i18n key) -->
    <nav class="sidebar-nav">
      <router-link to="/" exact-active-class="active">
        <IconGrid /> <span v-show="!collapsed">{{ t('nav.overview') }}</span>
      </router-link>
      <router-link to="/inventory" active-class="active"> … {{ t('nav.inventory') }} </router-link>
      <router-link to="/orders" active-class="active"> … {{ t('nav.orders') }} </router-link>
      <router-link to="/spending" active-class="active"> … {{ t('nav.finance') }} </router-link>
      <router-link to="/demand" active-class="active"> … {{ t('nav.demandForecast') }} </router-link>
      <router-link to="/reports" active-class="active"> … Reports </router-link>
    </nav>

    <!-- Footer: compact user / collapse toggle -->
    <div class="sidebar-footer"> … </div>
  </aside>
</template>
```

Use Vue Router's built-in `active-class="active"` / `exact-active-class` instead of the old manual `:class="{ active: $route.path === '/' }"` checks — cleaner and equivalent.

**Sidebar styling cues:**
- Fixed `width: var(--sidebar-w)`, `background: var(--sidebar-bg)`, full-height, `position: sticky; top: 0; align-self: flex-start` (or fixed with content margin).
- Nav links: each a flex row (`gap: var(--space-3)`), `padding: var(--space-2) var(--space-3)`, `border-radius: var(--radius-sm)`, `color: var(--sidebar-fg)`. Hover → lighter text + `--sidebar-active-bg`. Active → `--sidebar-fg-active` text on `--sidebar-active-bg` pill, optionally a left accent bar in `--color-accent`.
- Each link gets a small inline-SVG icon (24px) so the rail reads as a SaaS nav. Pick simple line icons (grid/dashboard, boxes/inventory, cart/orders, dollar/finance, chart/demand, document/reports).
- Group label (e.g. an uppercase `--text-subtle` "MENU" caption) above the nav list for polish.

**Responsive:** below ~1024px the sidebar collapses to `--sidebar-w-collapsed` (icons only, labels hidden via `v-show="!collapsed"`); below ~768px it becomes an off-canvas drawer toggled by a hamburger in the topbar. Keep it simple — a `collapsed` ref is enough for the icon-rail state.

## Step 3 — Polish the content area & shared components

Apply the tokens to the existing global classes (already in `App.vue`) and per-view styles. Checklist:

- **Content container:** `padding: var(--space-6)`, `max-width: var(--content-max)`, centered. Add a consistent **page header** block (title + subtle description) at the top of each view using `.page-header`.
- **Cards (`.card`, `.stat-card`):** `background: var(--surface)`, `border: 1px solid var(--border)`, `border-radius: var(--radius)`, `padding: var(--space-5)`, `box-shadow: var(--shadow-sm)`; hover lifts to `--shadow` and `--border-strong`. Uniform `--space-5` gaps in `.stats-grid`.
- **Tables:** keep the existing structure; ensure header uses `--text-muted`, uppercase, `letter-spacing`; row hover `--bg`; cell padding on the scale (`var(--space-2) var(--space-3)` → bump to `var(--space-3) var(--space-4)` for more air).
- **Badges:** map the existing `.badge.success/warning/danger/info` etc. to the `--*-soft` background + matching solid text token.
- **Buttons / interactive:** accent buttons use `--color-accent` → `--color-accent-hover`; consistent `--radius-sm`, `padding: var(--space-2) var(--space-4)`.
- **Typography rhythm:** page title `1.875rem/700`, card title `1.125rem/700`, body `0.938rem`, captions `0.75rem` uppercase muted. Section spacing uses `--space-5`.
- **Transitions:** keep them subtle (`0.15s–0.2s ease`) on color/shadow/background only.

## Step 4 — Execute via vue-expert

Hand `vue-expert` a single, explicit task containing: the token block (Step 1), the `Sidebar.vue` spec and new `App.vue` shell (Step 2), and the polish checklist (Step 3). Instruct it to:

1. Create `client/src/components/Sidebar.vue`.
2. Refactor `client/src/App.vue`: add the `:root` tokens, replace the `.top-nav` block with `<Sidebar />` + `.app-main` shell, relocate `LanguageSwitcher`/`ProfileMenu` into the slim topbar, keep `FilterBar` and both modals wired identically.
3. Convert remaining hardcoded colors/spacing in `App.vue` global styles to tokens.
4. Sweep `client/src/views/*.vue` and `client/src/components/*.vue` for obvious magic-number spacing/color and align to tokens — conservatively, without changing markup or logic.
5. Preserve every route, i18n key, prop, and emit.

Tell `vue-expert` **not** to touch `server/`, `api.js` contracts, or behavior.

## Step 5 — Verify

The frontend runs on `http://localhost:3000` (start it with the `start` skill if it isn't up). Verify — ideally via Playwright MCP (`vue-expert` can drive it), otherwise by opening the app:

- Sidebar renders on the left on every route; active link reflects the current page.
- All six nav destinations load and their data still renders (filters still work).
- `LanguageSwitcher` still toggles EN / 日本語 and labels still come from `t(...)`.
- `ProfileMenu` opens, and the profile + tasks modals still function.
- Layout holds at desktop and narrow widths (sidebar collapses, no horizontal overflow, `min-width:0` on `.app-main` prevents table blowout).
- No console errors.

Take before/after screenshots of the Dashboard if Playwright is available, and report what changed succinctly.

## Guardrails recap

- `.vue` edits → **always** `vue-expert`.
- Visual/structural only; no behavior, route, or API changes.
- Preserve i18n keys and all existing components.
- Spacing comes from the `--space-*` scale; colors from the token set.
- No emojis; no new UI frameworks.
- Document any non-obvious structural change with a brief comment in the code.
