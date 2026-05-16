# Evaluation suite — console-architecture

Checks that the agent grounds answers in **design decisions and project structure** for **Dataverse Console**: the mental model (React → client services → Fastify `/api/*` → controllers → services), the **right upstream** (Kubernetes CRDs, GitLab, Snowflake, Atlan), **Snowflake use-case `LocalCache`** (TTL aligned with **Snowflake data refresh cadence**) vs HTTP **`Cache-Control`**, **Snowflake auth split**, and **no `fetch` inside UI components**.

## Four pillars (rubric)

| Pillar | Question |
|--------|------------|
| **Outcome** | Correct layering and file areas; correct system of record (CRDs vs GitLab YAML vs Snowflake vs Atlan); Pathfinder OAuth vs pooled service JWT called out when Snowflake auth is in scope; Snowflake **server cache** tied to **use case** and **upstream update frequency** where relevant. |
| **Process** | Controllers delegate to services only; new work follows route → controller → service → client service + Redux pattern; new Snowflake features consider **dataset refresh cadence** before TTL/cache design. |
| **Style** | Uses **Console** / **Dataverse Console** vocabulary; references skill tables or `references/*.md` without contradicting **Gotchas** (offline `SERVE_JSON`, caching, secrets in env). |
| **Efficiency** | Concise pointers to `SKILL.md` “Where to look” / references; does not dump the whole stack or unrelated frameworks. |

## Artifacts

| File | Purpose |
|------|---------|
| [evals.json](evals.json) | Machine-readable eval cases (CI: `bash cli/run-evals.sh`); prompts and pillar pass signals per scenario |

## How to use

1. Pick a case from **`evals.json`** (`name` / `prompt`).
2. Run the agent with a prompt that should match this skill’s `description` (Console architecture, data tracing, Snowflake caching, or Fastify+Redux conventions).
3. Score **Outcome / Process / Style / Efficiency** where present for that case.
