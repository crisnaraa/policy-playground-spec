# Policy Playground — Component Dependency Map

## Dependency Overview

```
[ CloudFront + S3 ]
       │
  ┌────┴────────────────────────────────┐
  │           Shell Application         │
  │  (auth context, routing, layout)    │
  └────┬────────────────────────────────┘
       │ loads dynamically
  ┌────┼──────────────────┬──────────────────────┐
  │    │                  │                      │
  ▼    ▼                  ▼                      ▼
Policy Explorer  Simulation Studio   Results Dashboard   Admin Panel
   MFE               MFE                  MFE              MFE
   │                 │                    │                 │
   └────────────┬────┘                    │                 │
                │                         │                 │
                ▼                         ▼                 ▼
        [ Amazon API Gateway — REST — Cognito JWT Authorizer ]
                │
    ┌───────────┼───────────────────────────────────┐
    │           │              │                    │
    ▼           ▼              ▼                    ▼
Auth Service  Policy Service  Simulation Service  Results Service
    │              │                │                  │
    │           CodeCommit       ECS Task           Athena
    │           DynamoDB        (Job Executor)       S3
    │           S3                  │               DynamoDB
  Cognito                       Sandbox DB         (share tokens)
  Azure AD                      S3 (results)
                                DynamoDB (jobs)
                                Simulation Service (callback)

All services ──► SQS ──► Audit Service ──► S3 (audit lake) ──► Athena
```

---

## Detailed Dependency Table

| Component | Depends On | Dependency Type |
|---|---|---|
| Shell App | Cognito (auth tokens), CloudFront/S3 | Runtime |
| Policy Explorer MFE | Shell App (auth context), API Gateway | Runtime |
| Simulation Studio MFE | Shell App (auth context), API Gateway | Runtime |
| Results Dashboard MFE | Shell App (auth context), API Gateway | Runtime |
| Admin Panel MFE | Shell App (auth context), API Gateway | Runtime |
| API Gateway | Auth Service (JWT validation), ECS Services (downstream) | Runtime |
| Auth Service | AWS Cognito, Azure AD (SAML IdP), DynamoDB (sessions) | Runtime |
| Policy Service | CodeCommit (scripts), DynamoDB (metadata), S3 (scripts), CodePipeline (trigger) | Runtime |
| Simulation Service | ECS Task Runner, DynamoDB (jobs), Audit Service (SQS) | Runtime |
| Job Executor (ECS Task) | Simulation Service (config), Sandbox DB, S3 (write results), Simulation Service (callback) | Runtime |
| Results Service | Athena, S3 (result lake), DynamoDB (share tokens) | Runtime |
| Audit Service | SQS (event source), S3 (data lake), Athena | Runtime |
| All Services | CloudWatch (logs/metrics), ECR (images), SSM Parameter Store (secrets) | Operational |

---

## Shared Infrastructure Dependencies

```
ECR ──► all ECS services (container images)
SSM Parameter Store ──► Job Executor (sandbox credentials), all services (config)
CloudWatch ──► all ECS services, API Gateway (logging, alarms)
VPC / Security Groups ──► ECS cluster, Sandbox DB, RDS (network isolation)
```

---

## Key Architectural Boundaries

| Boundary | Description |
|---|---|
| **Frontend ↔ Backend** | All communication through API Gateway over HTTPS with JWT |
| **Simulation ↔ Sandbox** | Job Executor is the ONLY component with access to Sandbox DB |
| **Audit isolation** | Audit Service writes are append-only; no other service can delete audit records |
| **Session sharing** | Read-only share tokens have TTL; no write access for shared viewers |
| **Policy scripts** | Stored in CodeCommit; synced to S3 by Policy Service; never executed directly from Git |
