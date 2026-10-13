# Policy Playground — Components

## Overview
Policy Playground is structured as a micro-frontend architecture backed by containerized microservices on AWS ECS (Fargate). Each major feature domain has its own independently deployable SPA, all served via CloudFront. The backend exposes a unified REST API through Amazon API Gateway, with ECS Tasks handling policy job execution.

---

## Frontend Components

### 1. Shell Application (`shell-app`)
**Responsibility:** Root container / orchestrator for the micro-frontend ecosystem.
- Hosts the application shell (navigation, layout, auth context)
- Routes users to each feature SPA
- Manages shared authentication state (Cognito tokens)
- Served via CloudFront + S3

### 2. Policy Explorer MFE (`policy-explorer-mfe`)
**Responsibility:** Browse, search, and manage versioned policies from Git.
- Displays policy versions synced from AWS CodeCommit/CodePipeline
- Allows viewing diffs between versions
- Triggers policy execution (job submission)
- Served independently via CloudFront + S3

### 3. Simulation Studio MFE (`simulation-studio-mfe`)
**Responsibility:** Configure, launch, and monitor policy simulations.
- Build simulation parameters (target segments, date ranges, hypotheses)
- Submit jobs and poll job status endpoint
- Display live job progress indicator
- Served independently via CloudFront + S3

### 4. Results Dashboard MFE (`results-dashboard-mfe`)
**Responsibility:** Visualize and compare simulation outcomes.
- Display findings, metrics, and charts from Athena query results
- Support sharing sessions (read-only share link generation)
- Export reports (CSV, PDF)
- Served independently via CloudFront + S3

### 5. Admin Panel MFE (`admin-panel-mfe`)
**Responsibility:** User, sandbox, and access management.
- Manage user roles and permissions (via Cognito groups)
- Monitor sandbox data refresh schedules
- View audit logs from S3 data lake
- Served independently via CloudFront + S3

---

## Backend Components

### 6. API Gateway (`api-gateway`)
**Responsibility:** Single entry point for all frontend requests.
- REST API endpoints proxying to backend services on ECS
- JWT validation via Cognito Authorizer
- Rate limiting and throttling (10–50 concurrent users)
- Request routing to downstream services

### 7. Auth Service (`auth-service`)
**Responsibility:** Authentication and session management.
- SAML/OIDC federation with Azure AD via AWS Cognito Identity Pool
- Token issuance and refresh
- Role-based access control (analyst, admin, viewer)
- Runs on ECS Fargate

### 8. Policy Service (`policy-service`)
**Responsibility:** Policy version management and Git integration.
- Syncs policy versions from AWS CodeCommit via CodePipeline
- Exposes REST API for listing and retrieving policy versions
- Validates policy metadata (schema, active flag)
- Runs on ECS Fargate

### 9. Simulation Service (`simulation-service`)
**Responsibility:** Manages simulation lifecycle.
- Accepts job submission requests from the frontend
- Spawns ECS Tasks (ad hoc containers) for each simulation run
- Tracks job status in DynamoDB (PENDING → RUNNING → COMPLETED/FAILED)
- Exposes polling endpoint for job status
- Runs on ECS Fargate

### 10. Job Executor (`job-executor`)
**Responsibility:** Executes SQL/Python policy logic in an isolated container.
- Spun up as an on-demand ECS Task per simulation request
- Reads policy scripts from S3/CodeCommit
- Queries the dedicated Sandbox Database (RDS or DynamoDB)
- Writes results to S3 (Parquet/JSON) and publishes metadata to DynamoDB
- Terminates upon completion

### 11. Results Service (`results-service`)
**Responsibility:** Provides access to simulation results.
- Queries Athena over S3 result files
- Returns paginated findings to the frontend
- Generates shareable session tokens (stored in DynamoDB) for read-only access
- Runs on ECS Fargate

### 12. Audit Service (`audit-service`)
**Responsibility:** Append-only audit trail management.
- Writes all simulation events, user actions, and job metadata to S3 data lake (Parquet/JSON)
- Integrates with SQS for async audit event ingestion
- Runs on ECS Fargate

---

## Data Components

### 13. Policy Database (`policy-db` — DynamoDB)
**Responsibility:** Stores policy metadata, session state, findings, and hypotheses.
- Tables: `policies`, `sessions`, `simulation-jobs`, `findings`, `hypotheses`

### 14. Sandbox Database (`sandbox-db` — dedicated RDS/DynamoDB environment)
**Responsibility:** Always-on isolated environment with synthetic/anonymized data.
- Separate AWS account or VPC-isolated environment
- Used exclusively by Job Executor during simulations
- Refreshed on a scheduled basis

### 15. Audit Data Lake (`audit-lake` — S3)
**Responsibility:** Append-only audit storage.
- Stores all job results and audit events in Parquet/JSON
- Queried via Amazon Athena for results and reporting

### 16. Policy Repository (`policy-repo` — AWS CodeCommit + CodePipeline)
**Responsibility:** Version-controlled policy scripts (SQL/Python).
- CodeCommit as the Git remote for policy scripts
- CodePipeline triggers sync to Policy Service on every push
