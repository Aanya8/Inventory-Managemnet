---
name: saas-redesign
description: Redesigns a Vue 3 application's UI into a modern SaaS-style interface with a vertical navigation sidebar, consistent spacing, and a polished professional look.
---

# SaaS UI Redesign

Transform this Vue 3 application into a modern SaaS-style interface. Follow every step below in order.

---

## Step 1 — Audit the current layout

Before writing any code, read these files to understand what exists:
- `client/src/App.vue` — current layout shell, global styles, nav links
- `client/src/main.js` — routes and their paths
- Any composables used in App.vue (e.g. `useI18n`, `useAuth`, `useFilters`)

Extract:
1. Every `<router-link>` and its `to` path and label
2. Every global CSS class defined in App.vue's `<style>` block
3. Any components rendered in the header/nav (profile menus, language switchers, filter bars, etc.)
4. The color palette in use (note any CSS variables or hardcoded hex values)

---

## Step 2 — Design decisions to confirm before coding

Apply these defaults unless the existing codebase clearly contradicts them:

**Sidebar style:** Dark sidebar (#0f172a background, white text) with a lighter main content area (#f8fafc).

**Sidebar width:** 240px expanded. Do not implement a collapsed/icon-only state unless the app already has one — keep it simple.

**Layout structure:**
```
┌─────────────────────────────────────────────┐
│  Sidebar (240px fixed)  │  Main content     │
│  ┌─────────────────┐    │  ┌─────────────┐  │
│  │  Logo / Brand   │    │  │  Page area  │  │
│  ├─────────────────┤    │  │             │  │
│  │  Nav links      │    │  │             │  │
│  │  (with icons)   │    │  │             │  │
│  │                 │    │  │             │  │
│  ├─────────────────┤    │  │             │  │
│  │  Bottom section │    │  └─────────────┘  │
│  │  (user/profile) │    │                   │
│  └─────────────────┘    │                   │
└─────────────────────────────────────────────┘
```

**Top bar:** Remove the horizontal nav bar entirely. Move the logo/brand into the sidebar header. Move any utility components (language switcher, profile menu) to the sidebar bottom section.

**Filter bar:** If a `<FilterBar />` component exists, keep it in the main content area at the top of each page (not in the sidebar). It should sit just above `<router-view />`.

**Nav link style:**
- Default: transparent background, #94a3b8 text
- Hover: #1e293b background, #f1f5f9 text
- Active: #2563eb background, white text, subtle left border accent
- Include a simple SVG icon beside each label (use inline SVG paths, no icon library dependency)

**Spacing system:** Use these consistently —
- Sidebar padding: 1rem 0.75rem
- Main content padding: 1.5rem 2rem
- Card gap: 1.25rem
- Section margin-bottom: 1.5rem

---

## Step 3 — Rewrite App.vue

**MANDATORY: delegate this to the `vue-expert` subagent.** Pass it the full current contents of App.vue plus the complete spec below.

The new App.vue must:

1. **Remove** the `<header class="top-nav">` block entirely.

2. **Add** a sidebar as the first child of `.app`:
```html
<aside class="sidebar">
  <div class="sidebar-header">
    <!-- Brand logo / app name here -->
  </div>
  <nav class="sidebar-nav">
    <!-- One .nav-item per route -->
    <router-link class="nav-item" to="/" :class="{ active: $route.path === '/' }">
      <!-- inline SVG icon -->
      <span>Overview</span>
    </router-link>
    <!-- repeat for each route -->
  </nav>
  <div class="sidebar-footer">
    <!-- LanguageSwitcher and ProfileMenu here, stacked vertically -->
  </div>
</aside>
```

3. **Restructure** the main area:
```html
<div class="main-area">
  <FilterBar v-if="hasFilterBar" />   <!-- keep if it exists -->
  <main class="main-content">
    <router-view />
  </main>
</div>
```

4. **Change** `.app` to use `display: flex; flex-direction: row;` instead of column.

5. **Add scoped (or global) CSS** for the sidebar:
```css
.app {
  display: flex;
  flex-direction: row;
  min-height: 100vh;
}

.sidebar {
  width: 240px;
  min-height: 100vh;
  background: #0f172a;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  position: sticky;
  top: 0;
}

.sidebar-header {
  padding: 1.5rem 1rem 1rem;
  border-bottom: 1px solid #1e293b;
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  overflow-y: auto;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.625rem 0.875rem;
  border-radius: 8px;
  color: #94a3b8;
  text-decoration: none;
  font-size: 0.9rem;
  font-weight: 500;
  transition: background 0.15s, color 0.15s;
}

.nav-item:hover {
  background: #1e293b;
  color: #f1f5f9;
}

.nav-item.active {
  background: #2563eb;
  color: #ffffff;
}

.nav-item svg {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
}

.sidebar-footer {
  padding: 0.75rem;
  border-top: 1px solid #1e293b;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.main-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-width: 0; /* prevent flex overflow */
  background: #f8fafc;
}

.main-content {
  flex: 1;
  padding: 1.5rem 2rem;
  max-width: 1400px;
  width: 100%;
}
```

6. **Choose icons** for each nav item. Use simple, recognizable inline SVG paths (24x24 viewBox). Suggested icons per common route:
   - Overview / Dashboard: grid or home icon
   - Inventory: cube or box icon
   - Orders: clipboard or list icon
   - Spending / Finance: chart bar or currency icon
   - Demand: trending up icon
   - Reports: document chart icon
   - Restocking: arrow path / refresh icon

   Use Heroicons-style paths (outline variant). Do not import any icon library — write the SVG inline.

7. **Style the brand header** in `.sidebar-header`:
   - App name in white, font-weight 700, font-size 1.1rem
   - Subtitle in #64748b, font-size 0.75rem, margin-top 0.2rem

8. **Adapt ProfileMenu and LanguageSwitcher** for the dark sidebar context. If they have light-colored text assumptions, override with color: #cbd5e1 in the sidebar-footer context using a scoped descendant rule.

---

## Step 4 — Verify global styles don't break

After App.vue is rewritten, check that these global styles (which live in App.vue's `<style>` block) are still present and correct:
- `.card`, `.card-header`, `.card-title`
- `.stat-card`, `.stat-label`, `.stat-value`
- `.badge` and all variants (`.success`, `.warning`, `.danger`, `.info`)
- `.table-container`, `table`, `thead`, `th`, `td`
- `.loading`, `.error`
- `.page-header`, `h2`, `.page-header p`

These are used by every view. Do not remove them. The only things to remove are `.top-nav`, `.nav-container`, `.nav-tabs`, and `.nav-tabs a` rules — they are replaced by the sidebar.

---

## Step 5 — Remove max-width constraint from `.main-content`

The old layout used `max-width: 1600px` centered on a full-width page. With a 240px sidebar, adjust to `max-width: 1400px` and remove `margin: 0 auto` (the flex layout handles positioning now).

---

## Step 6 — Smoke test in browser

After the vue-expert completes the rewrite, use Playwright (`mcp__playwright__*`) to:
1. Navigate to `http://localhost:3000`
2. Take a screenshot and verify the sidebar is visible on the left
3. Click each nav link and confirm the active state updates and the page content changes
4. Confirm the main content area fills the remaining space correctly

If any nav link fails to show active state or the layout breaks, read the current App.vue and fix the issue.

---

## Key rules

- **NEVER** implement a collapsible/hamburger sidebar unless the user explicitly asks
- **NEVER** remove any existing functionality — only restructure layout and styles
- **ALWAYS** preserve all global CSS classes used by child views
- **ALWAYS** delegate `.vue` file writes/rewrites to the `vue-expert` subagent
- **DO NOT** add emojis to the UI
- **DO NOT** add new npm dependencies for icons or UI frameworks
