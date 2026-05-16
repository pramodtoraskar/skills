---
name: console-architecture
description: >-
  Use when you need structural or architectural context in the Console codebase: understanding layering and integrations, finding the right module for a concern, tracing which system owns data (Kubernetes, GitLab, Snowflake, Atlan, etc.), recording or interpreting design decisions, or adding features without fighting existing patterns. **Use when** designing new integrations, debugging cross-system failures, or implementing features spanning multiple data sources. **Do NOT use for** PatternFly UI component specifics, raw API implementation details, or deployment procedures—those have dedicated skills.
compatibility: Console codebase (Node.js/Express backend, React frontend, OpenShift/Kubernetes clusters, Snowflake analytics, GitLab integration, Atlan data catalog)
metadata:
  agentskills-spec: "1.0"
  version: "1.0.0"
  project: "Console"
---

# Console Architecture

The Console is a multi-tenant platform that provides unified visibility and management across Kubernetes clusters, analytics data (Snowflake), source control (GitLab), and data catalogs (Atlan). It operates as a web application with a Node.js/Express backend serving a React frontend, deployed on OpenShift clusters with service-account-based authentication to underlying systems.

The architecture follows a layered approach: frontend components consume REST APIs that orchestrate calls to external systems, with environment-specific authentication modes and caching strategies to handle both live and offline/demo scenarios.

## When to use

- "How do I add a feature that spans Kubernetes and Snowflake data?"
- "Where should I put logic that aggregates GitLab and cluster information?"
- "Which service owns user permissions vs. project metadata?"
- "How does authentication work between Console and the underlying systems?"
- "I'm getting 403 errors when accessing cluster resources—what controls access?"
- "Should this data come from Kubernetes CRs or Snowflake tables?"

## Mental model

```
Frontend (React/PatternFly)
    ↓ HTTP requests
API Layer (Express routes)
    ↓ authenticated calls
┌─────────────┬──────────────┬─────────────┬──────────────┐
│ Kubernetes  │  Snowflake   │   GitLab    │    Atlan     │
│ (via SA)    │ (JWT pools/  │ (OAuth)     │ (API keys)   │
│             │  OAuth)      │             │              │
└─────────────┴──────────────┴─────────────┴──────────────┘
         ↓            ↓            ↓            ↓
    Pod/CR data   Analytics   Repository    Data catalog
                 aggregates    metrics      lineage
```

## Default workflow

1. **Identify data ownership** - Check if the feature needs Kubernetes CRs, Snowflake aggregates, GitLab repository data, or Atlan catalog information
2. **Locate the service module** - Find or create the appropriate service in `src/server/services/` (kubernetes.service.js, snowflake.service.js, etc.)
3. **Define the API endpoint** - Add route handler in `src/server/routes/` with proper authentication context
4. **Implement authentication strategy** - Use ServiceAccount tokens for K8s, JWT pools or OAuth for Snowflake, OAuth for GitLab
5. **Add caching logic** - Set Cache-Control headers and implement LocalCache for expensive Snowflake queries if needed
6. **Handle offline mode** - Ensure SERVE_JSON mode works with fixture data in products.service.js
7. **Test RBAC boundaries** - Verify namespace-scoped permissions and cross-cluster access patterns
8. **Wire up frontend** - Connect React components to new API endpoints following existing patterns

## Where to look

| Path | Purpose | When to open |
|------|---------|--------------|
| `src/server/services/kubernetes.service.js` | K8s API calls, CR operations | Cluster data, pod status, custom resources |
| `src/server/services/snowflake.service.js` | Analytics queries, cost data | Usage metrics, aggregated reporting |
| `src/server/services/products.service.js` | Cross-system orchestration | Multi-source features, offline fixtures |
| `src/server/utils/config.js` | Environment configuration | Auth settings, external URLs, feature flags |
| `src/server/routes/` | API endpoint definitions | Adding new endpoints, auth middleware |
| `src/server/middleware/auth.js` | Authentication logic | Token handling, SSO integration |
| `config/environments/` | Per-env settings | NAMESPACE_INIT, KUBE_* variables |

## Stack and conventions

| Component | Technology | Authentication | Data Ownership |
|-----------|------------|----------------|----------------|
| Backend | Node.js/Express | JWT, ServiceAccount tokens | Orchestration only |
| Frontend | React, PatternFly | Session-based via backend | UI state |
| Kubernetes | OpenShift clusters | Pod ServiceAccount | Workload metadata, CRs |
| Snowflake | Cloud data warehouse | JWT pools (service), OAuth (Pathfinder) | Analytics, cost data |
| GitLab | Source control | OAuth tokens | Repository metrics |
| Atlan | Data catalog | API keys | Data lineage |
| Caching | LocalCache + HTTP headers | N/A | Temporary aggregates |

## Anti-patterns

**Kubernetes RBAC confusion**: In-cluster calls use the pod's ServiceAccount identity. Listing CRs across namespaces requires RBAC bindings for that SA on each target namespace. Check NAMESPACE_INIT, USERNAUT_NAMESPACE, and KUBE_* environment settings before debugging application code when seeing 403 errors.

**Mixed Snowflake auth modes**: Visualization and cost/agent-health endpoints use service account JWT pools per environment. Pathfinder features use end-user OAuth tokens refreshed via SSO (refreshSSOToken). These have different security boundaries and failure modes—don't mix them in the same request flow.

**Secrets in client code**: Never embed tokens, internal URLs, or credentials in skills, client bundles, or frontend code. Use server-side config via src/server/utils/config.js and environment variables.

**Cache-Control omission**: API handlers should set explicit Cache-Control headers per project convention. Don't omit caching headers without documenting the reason. This is separate from LocalCache used for Snowflake-backed aggregates.

**Offline mode gaps**: SERVE_JSON mode serves fixture data instead of live external system calls. Ensure new integrations have corresponding fixtures in products.service.js and related modules.

## Reference index

- `references/kubernetes-integration.md` - ServiceAccount auth, namespace scoping, CR operations
- `references/snowflake-auth-modes.md` - JWT pools vs OAuth, refreshSSOToken patterns
- `references/caching-strategies.md` - LocalCache vs HTTP headers, Snowflake query optimization
- `references/offline-fixtures.md` - SERVE_JSON mode, demo data patterns

## Evaluation suite

- `evals/README.md` - Architecture decision validation
- `evals/evals.json` - Cross-system integration tests
- `bash cli/run-evals.sh console-architecture`

## When this skill is not enough

For PatternFly-specific UI patterns and component usage, use the dedicated UI skills. For detailed API implementation and endpoint-specific logic, consult the API documentation in `docs/api/`. For deployment, scaling, and operational concerns, refer to the deployment guides in `docs/ops/`.