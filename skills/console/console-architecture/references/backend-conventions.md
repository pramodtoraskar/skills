# Console — backend conventions (server)

Checklist for Fastify/API work. Authoritative detail lives in **`CLAUDE.md`** (server section) and **`.cursor/rules/server-side.mdc`** — update those first when conventions change; keep this file as a short agent-facing map.

## Request path

```text
HTTP → route plugin (src/server/router/api/*.router.js)
     → controller (src/server/controllers/)
     → service (src/server/services/)
```

- **Register new API surfaces** on **`src/server/router/api/api.router.js`** (prefix per feature area).
- **Route files**: wire paths and schemas to handlers only — **no** heavy logic in route definitions beyond Fastify config.

## Services and integrations

- **Controllers** orchestrate; **services** own I/O (GitLab, Kubernetes, Snowflake, HTTP APIs, etc.).
- **One external concern per service module** where practical; reuse **`src/server/services/`** before adding duplicates.
- **OpenShift / Kubernetes**: the cluster client uses **in-cluster** credentials (workload **ServiceAccount**) when deployed; CR reads can span **several namespaces**—see **`SKILL.md` → OpenShift, Kubernetes API, and reading CRs** and keep **RBAC** in mind when adding new API calls.

## HTTP semantics

- **`Cache-Control`** on successful responses — choose `max-age` (or non-cache) based on how stale the data may be.
- **Snowflake / heavy aggregates**: the server also uses **use-case–scoped in-process cache** (`LocalCache` in `src/server/services/cache.service.js`, TTLs from `src/server/utils/config.js`). **Choose TTLs from how often the underlying Snowflake data is updated** for that dataset; new Snowflake queries should capture that before reusing vs adding a cache key. That layer is **not** a substitute for `Cache-Control`; it reduces upstream Snowflake load. See **`SKILL.md` → Snowflake queries and server-side cache**.
- Consistent JSON errors: **`{ error: true, message: '...' }`** with correct status codes; **no** stack traces or internal details to clients.

## Validation and logging

- **JSON Schema** on routes (`schema.body`, `schema.querystring`, `schema.params`) before controllers run.
- **`src/server/utils/logger.js`** for all logging — not **`console.log`**.
- Sonar-oriented JS preferences (e.g. optional chaining) per **`sonarqube-server-js.mdc`** where applicable.

## Security

- **`auth-check.plugin.js`** applies to authenticated routes; **public routes** are explicitly allowlisted in that plugin (new public endpoints must be added there when intentional).
- Treat all inputs as untrusted; validate at the route and again when building dynamic queries or shell usage.

## Configuration

- Environment-driven behavior via **`src/server/utils/config.js`** (and `.env` / deployment config) — never hardcode secrets in code.

## Verification after changes

`npm run format:check`, `npm run build`, `npm test` (per project workflow).
