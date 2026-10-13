# Policy Playground — Story-to-Unit Map

## Story Map Overview

| Story ID | Story Description | Unit |
|----------|------------------|------|
| US-001 | User can log in with corporate Azure AD account | `auth-and-identity`, `frontend-shell` |
| US-002 | Authenticated session persists across page refreshes | `auth-and-identity`, `frontend-shell` |
| US-003 | Collaborator can view an experiment session in read-only mode | `auth-and-identity`, `frontend-experiment-runner` |
| US-004 | Analyst can browse available credit policy versions | `policy-repository`, `frontend-policy-explorer` |
| US-005 | Analyst can view the full content of a specific policy version | `policy-repository`, `frontend-policy-explorer` |
| US-006 | Analyst can compare differences between two policy versions | `policy-repository`, `frontend-policy-explorer` |
| US-007 | Analyst can see the commit history for a policy branch | `policy-repository`, `frontend-policy-explorer` |
| US-008 | Analyst can configure and launch a policy experiment | `execution-engine`, `frontend-experiment-runner` |
| US-009 | Analyst can monitor real-time job execution status | `execution-engine`, `frontend-experiment-runner` |
| US-010 | Analyst can view execution logs for a completed job | `execution-engine`, `frontend-experiment-runner` |
| US-011 | System executes policy scripts as isolated ECS Fargate tasks | `execution-engine` |
| US-012 | Experiments run against an isolated sandbox with synthetic data | `sandbox-environment` |
| US-013 | Sandbox data can be refreshed/reset by an admin | `sandbox-environment` |
| US-014 | Analyst can explore findings from a completed experiment | `results-and-analytics`, `frontend-results-dashboard` |
| US-015 | Analyst can compare results across multiple policy versions | `results-and-analytics`, `frontend-results-dashboard` |
| US-016 | System maintains an append-only audit trail of all executions | `results-and-analytics`, `frontend-results-dashboard` |
| US-017 | User sees a unified navigation shell across all features | `frontend-shell` |
| US-018 | All infrastructure is defined as code (IaC) and version-controlled | `infrastructure-and-platform` |
| US-019 | CI/CD pipeline automatically builds and deploys on merge | `infrastructure-and-platform` |

---

## Unit → Stories Map

| Unit | Stories |
|------|---------|
| `infrastructure-and-platform` | US-018, US-019 |
| `auth-and-identity` | US-001, US-002, US-003 |
| `sandbox-environment` | US-012, US-013 |
| `policy-repository` | US-004, US-005, US-006, US-007 |
| `execution-engine` | US-008, US-009, US-010, US-011 |
| `results-and-analytics` | US-014, US-015, US-016 |
| `frontend-shell` | US-001, US-002, US-017 |
| `frontend-policy-explorer` | US-004, US-005, US-006, US-007 |
| `frontend-experiment-runner` | US-008, US-009, US-010, US-011, US-003 |
| `frontend-results-dashboard` | US-014, US-015, US-016 |

---

## Coverage Summary

- **Total Stories:** 19
- **Stories Fully Covered:** 19 (100%)
- **Units:** 10
- **Average Stories per Unit:** ~3.4
