# Preferred Frontend Stack Reference

> Last reviewed: 30 July 2026

## Purpose

This is the default frontend architecture for future projects backed by Django, Django REST Framework, or another independent API.

The guiding rule is:

> Start with the smallest architecture that meets the product requirements. Use React, Vite, TanStack Router, and TanStack Query by default. Start with TanStack Start only when server-side rendering or frontend server capabilities solve a confirmed business need.

This is a default, not a rule that overrides product requirements.

## Preferred Default Stack

| Technology | Responsibility |
| --- | --- |
| React | Component model, rendering, local interface state, and user interaction |
| TypeScript | Static type safety for components, routes, API contracts, forms, and domain models |
| Vite | Development server, hot-module replacement, environment variables, and production bundling |
| TanStack Router | Type-safe routing, nested layouts, search parameters, route guards, and code splitting |
| TanStack Query | Server-state fetching, caching, mutations, invalidation, retries, and background refetching |
| Tailwind CSS | Utility-first styling, responsive design, design tokens, and consistent visual rules |
| shadcn/ui | Project-owned component starting points that can be customized instead of treated as a black-box dependency |
| Radix UI | Accessible, headless primitives for dialogs, menus, selects, tooltips, tabs, and similar controls |
| React Hook Form | Efficient form state, field registration, validation feedback, and submission handling |
| Zod | Runtime validation and reusable, type-safe schemas for forms and client-side data boundaries |
| Vitest | Fast unit and integration testing that works naturally with Vite |
| React Testing Library | Testing React components through user-visible behavior rather than implementation details |
| Playwright | Browser-level testing of critical end-to-end workflows |

Optional additions should be justified by an actual requirement. Do not add a state library, form library, charting library, or animation framework merely because it is popular.

## Why This Is the Default

This stack provides:

- Fast development and production builds.
- Type-safe routes and navigation.
- Smooth navigation without full-page reloads.
- Reliable API caching and synchronization.
- A clear separation between the frontend and Django backend.
- Simple static deployment.
- A strong accessibility foundation.
- Testing at unit, component, integration, and browser levels.
- A practical migration path to TanStack Start if SSR becomes necessary.

It avoids introducing a frontend server before the product needs one.

## Application Data Flow

The normal request flow is:

```text
React page or component
        ↓
TanStack Query query or mutation
        ↓
Typed API module
        ↓
Shared Fetch client
        ↓
Django/independent API
        ↓
TanStack Query cache
        ↓
React re-renders the affected interface
```

### The No-Reload Experience

Smooth updates do not require TanStack Start. They come primarily from React and correctly configured TanStack Query.

After a successful mutation:

```text
User performs an action
        ↓
Backend processes it
        ↓
Mutation succeeds
        ↓
Related query keys are invalidated
        ↓
Fresh data is requested
        ↓
React updates the screen without a page reload
```

Use these mechanisms deliberately:

- Query invalidation after successful mutations.
- Optimistic updates only when rollback behavior is safe.
- Refetching when a route remounts or the browser regains focus.
- Polling only for data that genuinely needs periodic refreshes.
- Server-Sent Events or WebSockets for true server-pushed real-time updates.

Do not use aggressive polling to imitate real-time behavior when events or sockets are the correct solution.

## Initial Project Setup

These commands are a starting reference. Package requirements and scaffolding commands change, so verify the linked official documentation when creating a new project.

### 1. Create the React and TypeScript application

```bash
npm create vite@latest my-app -- --template react-ts
cd my-app
npm install
```

### 2. Add TanStack Router and Query

```bash
npm install @tanstack/react-router @tanstack/react-query
npm install -D @tanstack/router-plugin @tanstack/eslint-plugin-query
```

Use TanStack Router's file-based routing unless the project has a strong reason to use a code-defined route tree.

### 3. Add Tailwind CSS

Use the current Tailwind Vite installation instructions. The expected Tailwind 4 approach uses the Vite integration and imports Tailwind from the main stylesheet.

### 4. Add shadcn/ui

Initialize shadcn/ui for the existing Vite project, select a consistent visual preset, and add only the components the application uses.

Do not install an entire catalogue of unused components by default.

### 5. Add forms and validation

```bash
npm install react-hook-form zod @hookform/resolvers
```

Use React Hook Form and Zod for substantial forms. Small forms with one or two fields may use ordinary React state when that remains clearer.

### 6. Add component and integration testing

```bash
npm install -D vitest jsdom @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

### 7. Add end-to-end testing

```bash
npm init playwright@latest
```

Configure Playwright to start or connect to the development/test server and use a dedicated test backend or isolated test data.

## Recommended Project Structure

```text
src/
├── components/
│   ├── ui/                  # shadcn/Radix-based primitives
│   └── layouts/             # shared application layouts
├── features/
│   ├── auth/
│   ├── wallet/
│   ├── payments/
│   └── profile/
├── hooks/                   # reusable application hooks
├── lib/
│   ├── api/
│   │   ├── client.ts        # shared Fetch client
│   │   ├── types.ts
│   │   ├── auth.ts
│   │   └── feature-name.ts
│   ├── query-keys.ts
│   ├── validation/
│   └── utils.ts
├── routes/                  # TanStack file-based routes
├── test/
│   ├── setup.ts
│   └── helpers/
├── router.tsx
├── main.tsx
└── styles.css

e2e/
├── auth.spec.ts
├── critical-payment-flow.spec.ts
└── critical-user-journey.spec.ts
```

Prefer feature ownership over creating large global folders where unrelated code accumulates.

## API Integration Rules

### Shared client

Create one shared Fetch client responsible for:

- API base URL handling.
- JSON serialization and parsing.
- Authentication headers or cookie credentials.
- Normalized backend errors.
- Request cancellation through `AbortSignal`.
- Safe handling of unauthorized responses.
- Refreshing authentication only if the selected auth design requires it.

Do not repeat low-level `fetch` configuration across route components.

### Domain API modules

Group endpoint functions by domain:

```text
lib/api/auth.ts
lib/api/users.ts
lib/api/wallet.ts
lib/api/payments.ts
```

Components should call domain functions through TanStack Query instead of constructing URLs themselves.

### Query keys

Use predictable query keys:

```ts
["me"]
["wallet"]
["transactions", filters]
["user", userId]
["product", productId]
```

Invalidate the narrowest correct query after a mutation. Avoid invalidating every query in the application.

### Avoid scattered effect-based fetching

Prefer:

```ts
useQuery({
  queryKey: ["wallet"],
  queryFn: walletApi.getWallet,
});
```

Avoid using `useEffect` plus manually coordinated loading, data, and error state for normal server-data fetching.

## State Management Policy

Use:

- React state for temporary interface state.
- URL search parameters for shareable navigation and filter state.
- TanStack Query for remote/server state.
- React Hook Form for substantial form state.

Introduce another global state library only when a demonstrated client-state problem cannot be expressed clearly with these tools.

Do not store API data in Redux, Zustand, or Context merely to duplicate the TanStack Query cache.

## Form Policy

Use React Hook Form and Zod when a form has:

- Multiple related fields.
- Cross-field validation.
- Reusable schemas.
- Complex server validation errors.
- Dynamic or repeated fields.
- Multi-step submission.

Keep backend validation authoritative. Client validation improves user experience but must never replace Django validation.

Map backend errors to individual fields where possible, while preserving a form-level fallback message.

## Styling and Component Policy

- Define colours, spacing, radii, typography, and states as design tokens.
- Use Tailwind utilities consistently.
- Use shadcn/ui as editable source code, not as an untouchable dependency.
- Use Radix primitives when accessibility and interaction behavior are non-trivial.
- Prefer semantic HTML before reaching for a component abstraction.
- Add only components that the product currently needs.
- Verify keyboard access, focus visibility, labels, error messages, and touch target sizes.
- Keep product-specific components separate from generic UI primitives.

## Testing Strategy

### Vitest

Use Vitest for:

- Pure utility functions.
- Formatters and parsers.
- Zod schemas.
- Query-key factories.
- Business rules implemented in frontend code.

### React Testing Library

Use React Testing Library for:

- Form validation and submission behavior.
- Loading, empty, error, and success states.
- Accessible interaction with dialogs, menus, and inputs.
- Component behavior driven by mocked API boundaries.

Test what the user can observe rather than component internals.

### Playwright

Use Playwright for critical workflows such as:

- Registration and login.
- Password recovery.
- Permissions and protected routes.
- Checkout, deposit, or withdrawal flows.
- Important multi-page or redirect workflows.
- Regression-prone mobile layouts.

Do not attempt to cover every visual detail with end-to-end tests. Keep the E2E suite focused on journeys whose failure would materially affect users or the business.

## When to Start With TanStack Start

Start the project with TanStack Start when one or more of these is a confirmed requirement:

- Public pages need reliable SEO.
- The initial page should contain server-rendered content.
- Social-sharing metadata must vary by route or record.
- Server-side redirects or request middleware are required.
- The frontend needs server functions or frontend-owned API routes.
- Streaming or server-side route loading has a clear benefit.
- Authentication must be available during the initial server request.
- The product combines a substantial public website with an authenticated application.

TanStack Start provides full-document SSR, streaming, middleware, server functions, and client/server builds on top of TanStack Router and Vite.

### Strong examples

- E-commerce stores.
- Public marketplaces.
- Public booking platforms.
- Content-heavy products.
- SaaS products with important public acquisition pages.
- Public catalogues whose pages must be indexed.

## When Not to Start With TanStack Start

Prefer the default Vite stack when:

- The application is primarily authenticated.
- It is an internal dashboard, portal, HR system, or administrative tool.
- Django already owns the entire backend.
- SEO is not an important requirement.
- Static frontend deployment is desirable.
- The team does not need a frontend server.
- SSR would add operational work without a measurable user or business benefit.

Do not adopt SSR solely to get smooth, no-reload data updates. TanStack Query already provides that experience in a normal Vite application.

## Migration Readiness

A well-structured Vite application can adopt TanStack Start later.

To keep that migration manageable:

- Use TanStack Router from the beginning.
- Prefer file-based routes.
- Keep API access in dedicated modules.
- Use TanStack Query instead of scattered effect-based fetching.
- Isolate `window`, `document`, `localStorage`, and `navigator` access.
- Avoid browser-only work during component rendering.
- Avoid module-level mutable user or request state.
- Keep initial renders deterministic; do not directly render random or time-dependent values without controlling hydration.
- Separate reusable components from route entry components.
- Keep environment configuration explicit.
- Do not expose secrets through `VITE_` environment variables.

Introducing TanStack Start later will also change deployment from static files to client/server output when SSR is enabled.

## Authentication Decision

Choose authentication deliberately at project inception.

For purely browser-driven applications, bearer-token authentication can work with an independent Django API. For authenticated SSR, the frontend server must receive the user's authentication information with the request, which usually means a carefully designed secure cookie flow.

Do not assume that a token stored in `localStorage` is available during server rendering.

Authentication, refresh behavior, CSRF protection, CORS, cookie domains, and logout semantics must be designed together with the backend.

## Package and Dependency Policy

- Use one package manager and commit one authoritative lockfile.
- Pin production-sensitive tooling where reproducibility matters.
- Review dependency updates before merging them.
- Do not add packages for functionality already provided clearly by the platform or existing stack.
- Keep dev-only tools in `devDependencies`.
- Remove unused packages and generated component wrappers periodically.
- Never put secrets in `VITE_` variables; Vite client variables are bundled for browser access.

## Project Kickoff Checklist

Before implementation:

- [ ] Confirm whether the application is an SPA or needs SSR.
- [ ] Confirm whether public pages require SEO.
- [ ] Confirm the Django/API authentication design.
- [ ] Confirm the API base URLs for development, staging, and production.
- [ ] Define the API error-envelope convention.
- [ ] Establish route naming and access-control rules.
- [ ] Establish query-key conventions.
- [ ] Define design tokens and typography.
- [ ] Configure linting, formatting, and type checking.
- [ ] Configure Vitest and React Testing Library.
- [ ] Identify the critical Playwright workflows.
- [ ] Configure CI to run type checks, tests, linting, and production builds.
- [ ] Confirm the deployment target before relying on platform-specific behavior.

## Final Standard

### Default

```text
React
+ TypeScript
+ Vite
+ TanStack Router
+ TanStack Query
+ Tailwind CSS
+ shadcn/ui and Radix where appropriate
+ React Hook Form and Zod for complex forms
+ Vitest and React Testing Library
+ Playwright for critical end-to-end workflows
```

### Upgrade when justified

```text
Default stack
+ TanStack Start
```

The final decision rule is:

> Default to Vite. Begin with or migrate to TanStack Start when SSR, SEO, server middleware, server functions, or authenticated server rendering solves a confirmed requirement.

## Official References

- [Vite: Getting Started](https://vite.dev/guide/)
- [TanStack Router: Installation with Vite](https://tanstack.com/router/latest/docs/installation/with-vite)
- [TanStack Query: Installation](https://tanstack.com/query/latest/docs/framework/react/installation)
- [TanStack Start: Overview](https://tanstack.com/start/latest/docs/framework/react/overview)
- [Tailwind CSS: Documentation](https://tailwindcss.com/docs)
- [shadcn/ui: Vite Installation](https://ui.shadcn.com/docs/installation/vite)
- [React Hook Form: Get Started](https://react-hook-form.com/get-started)
- [Zod: Documentation](https://zod.dev/)
- [Vitest: Guide](https://vitest.dev/guide/)
- [Testing Library: React Testing Library](https://testing-library.com/docs/react-testing-library/intro/)
- [Playwright: Installation](https://playwright.dev/docs/intro)
