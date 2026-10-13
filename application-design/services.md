# Policy Playground — Services

## Architecture Style
Containerized microservices on **Amazon ECS (Fargate)** with a micro-frontend architecture on the client side.
All services are exposed through **Amazon API Gateway** (REST). Authentication is handled by **AWS Cognito** federated with **Azure AD (SAML/OIDC)**.

---

## AWS Infrastructure Services

| Service | AWS Resource | Purpose |
|---|---|---|
| Compute | Amazon ECS (Fargate) | Runs all backend microservices and ad hoc Job Executor tasks |
| API Gateway | Amazon API Gateway (REST) | Unified entry point; JWT auth via Cognito Authorizer |
| Authentication | AWS Cognito + Azure AD SAML Federation | Corporate SSO, token management, RBAC |
| Policy Version Control | AWS CodeCommit + CodePipeline | Git-integrated policy script versioning and sync |
| Job Queue | Amazon SQS | Async audit event ingestion; decouples Audit Service |
| Primary Database | Amazon DynamoDB | Stores sessions, policies, jobs, findings, hypotheses |
| Sandbox Database | Dedicated RDS/DynamoDB (isolated VPC) | Always-on anonymized/synthetic data environment for simulations |
| Results Storage | Amazon S3 + Amazon Athena | Append-only result and audit lake (Parquet/JSON); SQL query layer |
| Frontend Hosting | Amazon S3 + CloudFront | Hosts each micro-frontend SPA independently |
| Secrets Management | AWS Systems Manager (SSM Parameter Store) | Sandbox DB credentials, API keys |
| Monitoring | Amazon CloudWatch | Metrics, logs, alarms for ECS tasks and API Gateway |
| Container Registry | Amazon ECR | Stores Docker images for all ECS services and Job Executor |

---

## Application Services

### Auth Service
- **Runtime:** ECS Fargate (always-on)
- **Trigger:** API Gateway → `/auth/*`
- **Key integrations:** Cognito, Azure AD SAML IdP, DynamoDB (session state)
- **RBAC roles:** `analyst`, `admin`, `viewer`

### Policy Service
- **Runtime:** ECS Fargate (always-on)
- **Trigger:** API Gateway → `/policies/*` + CodePipeline event (via Lambda trigger → SQS)
- **Key integrations:** CodeCommit, DynamoDB (policy metadata), S3 (script storage)

### Simulation Service
- **Runtime:** ECS Fargate (always-on)
- **Trigger:** API Gateway → `/simulations/*`
- **Key integrations:** ECS Task Runner (spawns Job Executor), DynamoDB (job tracking)
- **Polling:** Frontend calls `GET /simulations/{jobId}/status` every 5 seconds

### Job Executor
- **Runtime:** ECS Task (on-demand, terminates when done)
- **Trigger:** Spawned by Simulation Service via `RunTask` API call
- **Key integrations:** Sandbox Database (RDS/DynamoDB), S3 (result output), Simulation Service (completion callback)
- **Isolation:** Runs in a separate ECS task definition with network access scoped to the sandbox VPC

### Results Service
- **Runtime:** ECS Fargate (always-on)
- **Trigger:** API Gateway → `/results/*`
- **Key integrations:** Amazon Athena, S3 (result lake), DynamoDB (share tokens)
- **Share model:** Generates time-limited, read-only tokens stored in DynamoDB

### Audit Service
- **Runtime:** ECS Fargate (always-on)
- **Trigger:** SQS queue (event-driven ingestion)
- **Key integrations:** SQS, S3 data lake, Athena
- **Pattern:** Append-only writes; never updates or deletes records

---

## Service Communication Patterns

| From | To | Protocol | Notes |
|---|---|---|---|
| Frontend MFEs | API Gateway | HTTPS REST | JWT Bearer token |
| API Gateway | Auth/Policy/Simulation/Results Services | HTTP (internal ALB) | ECS service discovery |
| Simulation Service | Job Executor | AWS SDK (`RunTask`) | Async ECS task spawn |
| Job Executor | Simulation Service | HTTPS callback | Reports completion |
| Any Service | Audit Service | SQS publish | Async, fire-and-forget |
| Audit Service | S3 | AWS SDK (PutObject) | Parquet/JSON append |
| Results/Admin | Athena | AWS SDK (StartQueryExecution) | SQL over S3 |
| Policy Service | CodeCommit | AWS SDK / HTTPS | Pull policy scripts |
| CodePipeline | Policy Service | Lambda trigger → SQS | Push notification on commit |
