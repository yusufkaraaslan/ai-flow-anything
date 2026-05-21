# Profile: Web Frontend

> **Purpose:** React, Vue, Angular, Svelte, and other frontend projects  
> **When to Use:** Browser-based applications and websites

---

## Description

Optimized for modern frontend development.

Supports:
- React (with hooks, context, Redux, etc.)
- Vue (2 or 3, Composition API, Options API)
- Angular (with RxJS, NgRx)
- Svelte/SvelteKit
- Solid, Preact, and others

---

## Detection Hints

**Indicators:**
- `package.json` exists
- `src/` with `.tsx`, `.jsx`, `.vue`, `.svelte`
- `vite.config.*`, `webpack.config.*`, `next.config.*`
- `index.html` or `public/` directory

**Confidence:**
- 3-4 matches: High confidence

---

## Capabilities

All 8 flows with frontend-specific patterns:
- Component architecture
- State management
- Routing
- Styling (CSS, SCSS, styled-components, Tailwind)
- API integration
- Accessibility

## Slots Discovered

- project-name, source-directory
- framework (React/Vue/Angular/Svelte)
- state-management (Redux/Zustand/Pinia/Vuex)
- styling (CSS/SCSS/Tailwind/styled-components)
- routing (React Router/Vue Router/Angular Router)
- testing (Jest/Vitest/Cypress/Playwright)

---

## Frontend-Specific Conventions

- Component composition
- Props down, events up
- Custom hooks/composables
- Lazy loading
- Accessibility (ARIA, keyboard nav)

See `rules.md` for complete convention list.
