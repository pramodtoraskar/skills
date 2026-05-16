# Console — data sources and integration map

Rough mapping from **HTTP API** → **server module** → **upstream system**. Paths are relative to `/api` unless noted.

## Core product data

| API prefix | Service / area | Upstream source of truth |
| --- | --- | --- |
| `/data-products` | `products.service.js`, controllers | **Kubernetes / OpenShift**: lists Dataverse **CRDs** per environment by calling the API from the **pod ServiceAccount** across **multiple namespaces** (operator namespaces from `NAMESPACE_INIT` + env suffix; see `k8s.service.js`). RBAC must allow list/get on those CRs. |
| `/kube` | Kubernetes routes/controllers | **Kubernetes API** (cluster operations used by the app). |
| `/gitlab` | `gitlab.service.js`, `gitlab-mr.service.js`, `dataproduct-config.service.js` | **GitLab HTTP API**: MRs, repository file content, `dataproduct-config` tree for product YAML. OAuth token from session when available; private token fallback for admin paths. |
| `/service-account` | GitLab + YAML utilities | **GitLab**: branches, commits, MRs for service account YAML; reads/writes tied to config repo. |
| `/github` | `github.service.js` | **GitHub public API** (e.g. release notes for repos). Unauthenticated read; 401 handled as auth required for higher rate limits if extended. |
| `/groups` | `groups.service.js` | **GitLab raw YAML** via configurable raw base URL (`DATA_PRODUCT_CONFIG_RAW_BASE_URL`) for group approval metadata and related group checks. |
| `/user/access/:email` | `usernaut.service.js` | **Usernaut HTTP API**: email → Dataverse developer/consumer group assets (service credentials). |
| `/user/gitlab/status`, `/user/gitlab/disconnect` | `user.controller.js` | **Server session only**: reads or clears `request.session.gitlab` (no external call for status beyond session). |
| `/user/ldap/:query`, `/user/ldap/uid/:uid` | `ldap.js` | **Corporate LDAP**: search and resolve `cn` by uid for forms and lookups. |
| `/cache` | `cache.service.js` | Application cache layer (invalidation and keys—not an external business source). |

## Kubernetes CRs vs dataproduct-config (Git)

| | **Kubernetes / OpenShift CRs** | **`dataproduct-config` (GitLab repo)** |
| --- | --- | --- |
| **What it represents** | **Live** cluster state: what is reconciled, installed, and visible to the operator **right now** | **Declarative** intent: YAML under review, promoted through Git, templates and metadata |
| **Typical use in Console** | Product **inventory**, status, runtime fields, per-environment instances | Forms, MR-driven changes, service-account YAML trees, automation that edits files |
| **Why both exist** | Operators and namespaces are the **runtime** source of truth for “what exists” | Git is the **governance and change** path for “what we want next” |

Console reads **CRs from the API** when the UI needs **live** data; it reads **Git** when the workflow is **config-as-code** (MRs, file paths, branches) rather than live reconciliation state. See also **decisions** in [references/decisions.md](decisions.md) (*Data source strategy*).

## Data product names and descriptions

Product **instances** still come from **Kubernetes CRDs** (`/data-products/products`). Friendly titles and long-form copy come from **synced JSON on disk** plus optional fields on the CR:

| What | How it reaches the UI | Source of truth |
| --- | --- | --- |
| **Display title** (e.g. page H1) | `GET /data-products/attributes` → Redux `productAttributes`; **`getProductDisplayName()`** maps slug → `displayName` | **`synced-repo/displayNameMapping.json`** (served as JSON by `handleGetDataProductsAttributes`). |
| **Overview / narrative description** (product detail page) | `products.service.js` merges **`cruft.json`** into each product as **`product.cruft`**; `dp-description.page.jsx` shows **`topEnvSpec.description`** if set, else **`cruft.context.cookiecutter.team_description`** | **Kubernetes**: **`spec.description`** on the **top environment** CR when present; **fallback**: **`synced-repo/cruft.json`** (cookiecutter **`team_description`**) keyed by product name. |
| **AI-ready properties + description** | `GET /data-products/ai-ready` | **`synced-repo/properties.json`** merged with **`synced-repo/cruft.json`**; controller sets each product’s **`description`** from **`cruft[product].context.cookiecutter.team_description`**. |

The **`synced-repo/`** directory is populated by deployment/sync (not written by ad-hoc Console requests); treat it as **generated config** from your dataproduct / cookiecutter pipeline.

## Metadata and governance

| API prefix | Service / area | Upstream |
| --- | --- | --- |
| `/atlan` | `common/atlan/sdk.js`, controllers | **Atlan** (`redhat.atlan.com` search/index API) with bearer token from env. Used for metadata such as marts/schemas tied to data products. |

## Snowflake and analytics

| API prefix | Service / area | Upstream |
| --- | --- | --- |
| `/visualization/*` | `controllers/visualization.controller.js` + `common/snowflake/*` | **Snowflake**: cost views (e.g. cost ops, Fivetran, Snowpipe), agent health tables, InOrbit-style health queries—via **JWT connection pools** per environment (`snowflake-service.js`). |
| `/pathfinder` | `pathfinder.service.js` | **Snowflake** Pathfinder database/views. Uses **user OAuth token** (`OAUTH` authenticator), not the pooled service JWTs. Optional local JSON via `PATHFINDER_USE_LOCAL_DATA`. |

Submodules under `src/server/services/common/snowflake/`:

- `agent-health/` — AI/agent operational metrics.
- `costops/` — Snowflake/Fivetran/cost reporting views.
- `inorbit/` — environment health style datasets.

## Supporting infrastructure services

| Module | Role |
| --- | --- |
| `vault.service.js` | HashiCorp Vault integration where configured. |
| `privatebin.service.js` | PrivateBin paste/API flows. |
| `key-sharing.service.js` | Key sharing workflows (paired with crypto/email as implemented). |
| `ldap.js` | LDAP directory lookups when enabled. |
| `common/email/*` | Outbound email (templates + transport). |

## User identity — where details come from

Identity is **not** a single API; it is layered:

| Layer | Source | What the app gets |
| --- | --- | --- |
| **Login / SSO** | **Red Hat SSO (OIDC)** via `auth.plugin.js` | OIDC claims populate **`request.session.user`** (e.g. `email`, `preferred_username`, name fields). |
| **LDAP enrichment** | **`ldap-add.plugin.js`** on HTML navigations | Uses `preferred_username` → **`ldap.js`** (`ldapUserDetails`) → adds **`ldapGroups`**, **`ldapManager`** on **`session.user`** (skipped for `/api/*` and static asset paths). |
| **Bootstrap into React** | **`client.router.js`** HTML shell | **`globalThis.USER_DATA`** is set to **`JSON.stringify(request.session.user)`** so the SPA starts with the same identity + LDAP fields. **`src/client/store/slices/user.slice.jsx`** initializes Redux `user` from **`globalThis.USER_DATA`**. |
| **GitLab profile** | **GitLab OAuth** session | **`request.session.gitlab`** holds tokens and **`userInfo`**; client loads GitLab-specific info via **`GET /api/user/gitlab/status`** (`user.controller.js`). |
| **Dataverse entitlements** | **Usernaut** | **`GET /api/user/access/:email`** → developer/consumer assets for that email (used where product access must be explicit). |
| **Directory lookups** | **LDAP** | **`/api/user/ldap/...`** for typeahead / resolving **`cn`** by uid (`ldap.js`). |

So: **who you are** comes from **SSO + session** (and LDAP additions on page load); **what you can access in Dataverse groups** for some flows comes from **Usernaut**; **GitLab-facing identity** comes from the **GitLab-linked session**.

## Authentication (session gates)

| Mechanism | Purpose |
| --- | --- |
| Red Hat SSO (OIDC) | Primary identity in **`request.session.user`**. |
| GitLab OAuth | **`request.session.gitlab`** for **user-attributed** GitLab API actions (MRs, file API). |
| `AUTH_ENABLED=false` | Dev user injection (see `auth-check.plugin.js`). |

Public routes bypass login (see `auth-check.plugin.js`), including `/api/ai-ready` and specific webhooks.

## Client wiring

Browser code calls Fastify via relative `/api/...` URLs through **`src/client/services/*.jsx`** thunks. Redux slices live under `src/client/store/`. Initial **logged-in user** fields for the SPA come from **`globalThis.USER_DATA`** (server-rendered), not from a dedicated “who am I” JSON API on first paint. No direct Snowflake/Kubernetes/GitLab calls from the browser for privileged operations—always through the API.
