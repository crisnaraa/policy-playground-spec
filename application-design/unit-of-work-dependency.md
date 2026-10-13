# Policy Playground — Unit Dependency Map

## Dependency Graph

```
infrastructure-and-platform  (no dependencies — foundational)
        │
        ▼
auth-and-identity
        │
        ├──────────────────────────────────┐
        │                                  │
        ▼                                  ▼
sandbox-environment              policy-repository
        │                                  │
        └──────────┬───────────────────────┘
                   │
                   ▼
           execution-engine
                   │
                   ▼
          results-and-analytics
                   
frontend-shell ──── depends on ──── auth-and-identity
        │
        ├── frontend-policy-explorer ─────── policy-repository
        ├── frontend-experiment-runner ───── execution-engine
        └── frontend-results-dashboard ───── results-and-analytics
```

---

## Dependency Table

| Unit | Depends On |
|------|-----------|
| `infrastructure-and-platform` | *(none — foundational)* |
| `auth-and-identity` | `infrastructure-and-platform` |
| `sandbox-environment` | `auth-and-identity` |
| `policy-repository` | `auth-and-identity` |
| `execution-engine` | `auth-and-identity`, `policy-repository`, `sandbox-environment` |
| `results-and-analytics` | `execution-engine`, `auth-and-identity` |
| `frontend-shell` | `auth-and-identity` |
| `frontend-policy-explorer` | `frontend-shell`, `policy-repository` |
| `frontend-experiment-runner` | `frontend-shell`, `execution-engine` |
| `frontend-results-dashboard` | `frontend-shell`, `results-and-analytics` |

---

## Implementation Order (Topological Sort)

### Phase 1 — Platform Foundation
1. `infrastructure-and-platform`

### Phase 2 — Core Services
2. `auth-and-identity`
3. `sandbox-environment` *(parallel with policy-repository)*
4. `policy-repository` *(parallel with sandbox-environment)*

### Phase 3 — Orchestration & Analytics
5. `execution-engine`
6. `results-and-analytics`

### Phase 4 — Frontend
7. `frontend-shell`
8. `frontend-policy-explorer` *(parallel)*
9. `frontend-experiment-runner` *(parallel)*
10. `frontend-results-dashboard` *(parallel)*
