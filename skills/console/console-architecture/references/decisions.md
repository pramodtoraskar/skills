# Console — architectural decisions

Concise, decision-focused notes for agents implementing features. Pair with code and `README.md` / `CLAUDE.md` for full conventions.

## Platform and layering

1. **Backend-for-frontend**: The React app talks to **Fastify only**. Privileged access to Snowflake, Kubernetes, and GitLab stays server-side.
2. **Controllers stay thin**: Route handlers orchestrate; **all** third-party and DB access goes through **`src/server/services/`** modules.
3. **One concern per service file**: e.g. GitLab vs Kubernetes vs Snowflake paths stay separated for maintainability.
4. **Input validation at the edge**: Prefer Fastify JSON Schema on routes before controller logic.

## Client state and API usage

5. **Redux as the UI cache**: Client **`fetch`** lives in **service modules** (`src/client/services/*.jsx`); updates go through **`createAsyncThunk`** and reducers—components **select** state, no inline fetch in views.
6. **Native fetch only** on the client (no axios).

## Identity and audit

7. **Dual session**: **SSO** for user identity; **GitLab OAuth** for GitLab API calls that must attribute to the user (e.g. MR creation). Private tokens exist for **fallback/admin** automation only where implemented.
8. **Service account requests** encode identity in branch naming and Git history (Console repository conventions—see `README.md`).

## Data source strategy

9. **Runtime inventory from Kubernetes**: Live product instances come from **CRDs** per environment/namespace, not only from Git. On **OpenShift**, the API server uses the **pod ServiceAccount** (`KubeConfig.loadFromDefault`); namespaces are built from **`NAMESPACE_INIT`** + per-env suffix, with **Usernaut** CRs in **`USERNAUT_NAMESPACE`**. **RBAC** must allow that ServiceAccount to **list/get** CRs in every namespace the app reads—treat **403** as a cluster binding or config issue until proven otherwise.
10. **Declarative desired state in Git**: **`dataproduct-config`** repository defines YAML for domains, groups, and product structure; GitLab API is used to read/write for forms and automation. This is **declarative / review-centric** data—**not** a substitute for **live** product inventory from **cluster CRs** when the UI must show what is actually reconciled (see **Kubernetes CRs vs dataproduct-config (Git)** in [references/data-sources.md](data-sources.md)).
11. **Snowflake**: Operational/analytical datasets (cost, agent health, etc.) use **service JWT pools** per environment. **Pathfinder** intentionally uses the **user’s Snowflake OAuth** session so access respects Snowflake RBAC for that user. **Server cache TTLs** (`LocalCache`, env in `config.js`) should align with **how often each dataset is updated in Snowflake**; new Snowflake-backed features must document that cadence before choosing reuse vs a new cache key.
12. **Atlan** for enterprise metadata (e.g. mart/schema discovery) via HTTP API, separate from Snowflake execution.

## Operational behavior

13. **Explicit HTTP caching**: Successful API responses should include appropriate **`Cache-Control`** headers (vary by volatility of data).
14. **Fixture mode**: `SERVE_JSON` and related flags allow running UI against **JSON fixtures** without live clusters—used for demos and parts of CI/dev.
15. **Logging**: Server uses structured **`logger`**—not raw `console.log` in production paths (existing violations should be fixed opportunistically).

## UX architecture

16. **PatternFly v6** as the UI standard; design tokens and density rules in **`docs/design-rules.md`** for consistency with “Dataverse Console” product language.

## Frontend organization

17. **Route-centric composition**: **`pages/`** holds routed screens and screen-level orchestration (Redux wiring, params); **`components/`** holds reusable and feature UI; **`sections/`** holds larger cross-page blocks; **`hooks/`** and **`utils/`** hold shared behavior vs pure helpers—see [references/frontend-conventions.md](frontend-conventions.md). This is **not** MVC with separate controllers; orchestration lives on pages (and hooks where reused).

## Identity on the client

18. **Server-driven user bootstrap**: The HTML shell sets **`globalThis.USER_DATA`** from **`request.session.user`** so Redux initializes without a separate client-side “session fetch”. LDAP groups/manager are attached server-side (**`ldap-add.plugin.js`**) before that serialization where applicable—see section *User identity — where details come from* in [references/data-sources.md](data-sources.md).

---

## How to record a new decision

When a choice affects future features (e.g. “we read X from Atlan instead of Snowflake for reason Y”), add a numbered bullet with **context**, **decision**, and **consequence** (one short paragraph). Optionally link the PR or ADR if the team maintains one elsewhere.
