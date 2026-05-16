# Console — frontend conventions (client)

Checklist for React/UI work. Authoritative detail lives in **`CLAUDE.md`** (client section) and **`.cursor/rules/client-side.mdc`** — update those first when conventions change; keep this file as a short agent-facing map.

## Layout and stack

| Topic | Convention |
| --- | --- |
| UI kit | PatternFly v6 only (`@patternfly/react-core`, `@patternfly/react-table`, `@patternfly/react-templates`, `@patternfly/react-topology`, `@patternfly/react-charts`) |
| Styling | Prefer PF utility classes and component props; minimal custom CSS |
| Build | Vite; entry wiring per existing app structure |

## Folder structure (what goes where)

Console uses **route-centric composition**: each navigable screen under **`pages/`** wires routing, loads data via Redux (dispatch/selectors), and **composes** smaller pieces. That is **not** classical MVC “controllers”—there is no separate controller layer. It closest matches **route-level (smart) containers** plus **presentational / feature components**, sometimes called a **container vs presentational** split at the router boundary.

| Path | Responsibility |
| --- | --- |
| **`src/client/pages/`** | **Route screens** (often `*.page.jsx`): owns screen-level logic—route params, `dispatch` / `useSelector`, loading/error UX for that route, and composing child UI. Put **page-specific orchestration** here; keep files focused and delegate repeated UI to `components/` or `sections/`. Subfolders mirror areas (e.g. `data-products/`, `cost-ops/`, `access-screens/`). |
| **`src/client/components/`** | **Reusable UI**—feature modules (`product-detail/`, `costops/`, …), shared primitives (`lib/`—tooltips, ErrorBoundary, dropdowns), chrome (`layout/`, `masthead`), charts (`charts/`). Prefer **props down**; avoid embedding route assumptions unless the component is clearly route-scoped. |
| **`src/client/sections/`** | **Large composed blocks** shared across pages (e.g. cost-ops overview tiles/charts). Use when a slice of UI is bigger than a single card but not tied to one route file. |
| **`src/client/layouts/`** | **Shared layout wrappers** (shells around multiple pages or variants). |
| **`src/client/hooks/`** | **Custom hooks** (`*.hook.jsx` or similar)—reusable **stateful** behavior: URL/query sync, table state, LDAP typeahead, access-form plumbing. Prefer hooks when two or more screens/components share the same interaction logic. |
| **`src/client/utils/`** | **Pure helpers**—formatting, validation, schema/pathfinder helpers, shared constants modules. Stateless functions and small modules **without** React lifecycle; name often reflects domain (`product-validation.jsx`, `schema-helpers.jsx`). Files may be named **`*-helpers.jsx`**; there is **no** separate top-level `helpers/` folder—use **`utils/`** (or rare co-location next to a feature). |
| **`src/client/services/`** | **HTTP client only**—`fetch` wrappers for `/api`; no UI logic. |
| **`src/client/store/`** | Redux **`slices/`** and **`selectors/`**—global client state derived from API data. |
| **`src/client/assets/`** | Static assets (images, etc.). |

Tests mirror source layout under **`src/client/test/`** (`test/pages/`, `test/components/`, `test/hooks/`, …).

## Data flow (required pattern)

1. **`src/client/services/*.jsx`** — `fetch` to `/api/...` only here (native `fetch`, no axios).
2. **Redux slice** — `createAsyncThunk` wraps service calls; **`extraReducers`** handle `pending` / `fulfilled` / `rejected`.
3. **Pages (and route-level parents)** — `dispatch(thunk)` + `useSelector`; **no** `fetch` inside leaf components for shared server data. **`fetch` stays in `services/`**; pages orchestrate when thunks run (e.g. `useEffect` + dispatch by route or tab).

Store registration: **`src/client/store/index.store.jsx`** — add new slice reducers here when introducing a new domain slice.

## Errors and UX

- Wrap page/section boundaries with **`ErrorBoundary`** (`src/client/components/lib/ErrorBoundary.jsx`) where appropriate.
- Thunks must surface failures: loading states + user-visible errors (no silent rejects).

## Layout gotchas (PatternFly)

From client-side rules — common mistakes:

- Avoid **`Flex direction="column"` + `FlexItem`** for vertical page flow (stretch/whitespace issues). Prefer fragments or `<div>` with explicit spacing.
- When swapping views in one container, use a **`key`** on the active view so layout remeasures.
- Do not add extra **`PageSection`** only for spacing (heavy padding); prefer a light `<div>` when nesting inside an existing page shell.

## Performance

- **`React.lazy` + `Suspense`** for route-level splitting.
- **`React.memo` / `useMemo` / `useCallback`** for lists and heavy subtrees.
- Do **not** pass **new object/array/function literals** inline into props of **memoized** children.

## Design reference

- **`docs/design-rules.md`** — spacing, typography, tokens (“Dataverse Console” product language).

## Testing

- Jest + React Testing Library; follow **`jest/prefer-to-have-length`** and avoid duplicate **`describe`** titles in the same parent.

## Verification after changes

`npm run format:check`, `npm run build`, `npm test` (per project workflow).
