# Policy Playground — Application Design

## Executive Summary

**Policy Playground** is an internal analytics platform that enables credit policy analysts to simulate, compare, and iterate on credit policy versions in a safe sandbox environment — without touching production data. The platform is built entirely on AWS with a micro-frontend architecture on the client side and containerized microservices on ECS Fargate for the backend.

---

## Architecture Style

| Dimension | Decision |
|---|---|
| **Frontend** | Micro-frontend (independent SPAs per feature, CloudFront + S3) |
| **Backend** | Containerized microservices on ECS Fargate |
| **API Layer** | Amazon API Gateway (REST) with Cognito JWT Authorizer |
| **Authentication** | AWS Cognito + Azure AD federation (SAML/OIDC) |
| **Job Execution** | On-demand ECS Tasks (ad hoc containers per simulation) |
| **Policy Versioning** | AWS CodeCommit + CodePipeline (Git-integrated) |
| **Results Delivery** | Polling (frontend polls `/status` endpoint every ~5 seconds) |
| **Collaboration** | Shared sessions with read-only access (time-limited share tokens) |
| **Sandbox Data** | Always-on dedicated environment (isolated RDS/DynamoDB VPC) |
| **Audit** | Append-only S3 data lake (Parquet/JSON) queried via Athena |
| **Concurrency Target** | 10–50 concurrent users (medium team scale) |

---

## System Context

```
   [Corporate Users: Analysts, Admins, Viewers]
              │ HTTPS (browser)
              ▼
      [ CloudFront + S3 ]
       (Micro-Frontend SPAs)
              │ HTTPS + JWT
              ▼
      [ Amazon API Gateway ]
              │
    ┌─────────┼──────────────┬──────────────┐
    ▼         ▼              ▼              ▼
Auth      Policy         Simulation     Results
Service   Service         Service        Service
(Cognito) (CodeCommit)   (ECS Tasks)    (Athena)
              │                │              │
            DynamoDB    Sandbox DB         S3 + Athena
                        (isolated)
              └───────── All ───────────────┘
                              │
                         Audit Service
                        (SQS → S3 lake)
```

---

## Component Summary

### Frontend (Micro-Frontend SPAs)
| Component | Feature Domain |
|---|---|
| Shell Application | Auth context, routing, shared layout |
| Policy Explorer MFE | Browse & compare versioned policies from Git |
| Simulation Studio MFE | Configure, launch & monitor simulations |
| Results Dashboard MFE | Visualize findings, compare runs, share sessions |
| Admin Panel MFE | User management, sandbox health, audit log viewer |

### Backend (ECS Fargate Services)
| Component | Responsibility |
|---|---|
| Auth Service | SAML federation, token issuance, RBAC |
| Policy Service | Git sync (CodeCommit), policy metadata management |
| Simulation Service | Job lifecycle management, ECS task orchestration |
| Job Executor (ECS Task) | Isolated policy script execution against sandbox data |
| Results Service | Athena queries, result delivery, share token management |
| Audit Service | Append-only event ingestion (SQS → S3) |

### Data Stores
| Store | Technology | Purpose |
|---|---|---|
| Policy & Session DB | DynamoDB | Sessions, jobs, findings, hypotheses, policy metadata |
| Sandbox DB | Dedicated RDS/DynamoDB (isolated VPC) | Synthetic/anonymized data for simulations |
| Audit Data Lake | S3 (Parquet/JSON) | Append-only events and job results |
| Policy Scripts | S3 + CodeCommit | Version-controlled SQL/Python policies |

---

## Key Design Decisions & Rationale

### 1. ECS Tasks for Job Execution
Each policy simulation spawns an isolated ECS Task container. This provides:
- **Resource isolation** per simulation run
- **Language flexibility** (SQL via JDBC, Python in-process)
- **Automatic cleanup** (container terminates on completion)
- **Cost efficiency** (Fargate pay-per-use)

### 2. Polling for Status
The frontend polls `GET /simulations/{jobId}/status` every 5 seconds. Given medium team scale (≤50 users) and the expected duration of simulations (minutes), polling is appropriate and avoids WebSocket infrastructure complexity.

### 3. CodeCommit for Policy Versioning
AWS CodeCommit provides a fully managed, IAM-integrated Git service. CodePipeline triggers Policy Service syncs on every push, keeping policy metadata in DynamoDB up to date without requiring agents to pull on demand.

### 4. Dedicated Sandbox Environment
The sandbox database is always-on and VPC-isolated. Only the Job Executor ECS Task has network access to it. This prevents accidental production data exposure and enables safe, reproducible simulation runs.

### 5. Read-Only Session Sharing
Share tokens (DynamoDB items with TTL) allow analysts to share simulation results with stakeholders without giving them write access. Tokens are scoped to a session and expire automatically.

### 6. Micro-Frontend Architecture
Each feature (Policy Explorer, Simulation Studio, Results Dashboard, Admin) is independently deployable, allowing teams to update individual features without full-platform redeployments.

---

## Security Model

| Concern | Approach |
|---|---|
| Authentication | Cognito + Azure AD SAML federation |
| Authorization | RBAC via Cognito groups (`analyst`, `admin`, `viewer`) |
| API Security | JWT validation on every request via Cognito Authorizer |
| Sandbox Isolation | Job Executor is the only service with VPC route to Sandbox DB |
| Audit Integrity | Append-only S3 writes; no delete permissions on audit bucket |
| Secrets | SSM Parameter Store for DB credentials and API keys |
| Network | ECS services in private subnets; API Gateway as only public ingress |

---

## Deployment Topology

```
us-east-1 (primary region)
├── VPC
│   ├── Public subnets: ALB, NAT Gateway
│   ├── Private subnets: ECS Fargate cluster
│   └── Isolated subnets: Sandbox DB (RDS/DynamoDB)
├── CloudFront distributions: Shell + 4 MFE SPAs
├── API Gateway: REST API + Cognito Authorizer
├── DynamoDB tables: policies, sessions, jobs, findings
├── S3 buckets: policy-scripts, job-results, audit-lake, mfe-assets
├── CodeCommit: policy-scripts repo
├── CodePipeline: policy-sync pipeline
├── SQS queues: audit-events
├── ECR: service images
└── CloudWatch: logs, metrics, alarms
```
