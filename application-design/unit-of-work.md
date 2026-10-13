# Policy Playground — Units of Work

## Overview

The system is decomposed into **10 Units of Work** across three layers:
- **Platform Layer**: Infrastructure & Auth
- **Backend Services Layer**: Policy Repository, Execution Engine, Sandbox, Results & Analytics
- **Frontend Layer**: Shell + 3 Micro-Frontend SPAs

---

## Unit 1 — Infrastructure & Platform

**Name:** `infrastructure-and-platform`

**Boundaries:** All AWS infrastructure definitions (IaC), networking, CI/CD pipelines, and platform-level configuration shared across all units.

**Responsibilities:**
- AWS CDK / CloudFormation IaC for all resources
- VPC, subnets, security groups, and NAT gateway
- ECS cluster and Fargate task definitions
- S3 buckets (audit lake, result store, logs)
- DynamoDB table definitions
- Athena workgroup and Glue catalog
- CodeCommit repository and IAM policies
- CI/CD pipeline (AWS CodePipeline + CodeBuild)
- CloudWatch logging, alarms, and dashboards

**Stories:** US-018, US-019

**Dependencies:** None (foundational unit)

---

## Unit 2 — Auth & Identity

**Name:** `auth-and-identity`

**Boundaries:** Handles all authentication and authorization concerns. Integrates Azure AD via SAML federation through AWS Cognito. Issues JWT tokens consumed by all other services.

**Responsibilities:**
- AWS Cognito User Pool configuration with SAML IdP (Azure AD)
- JWT token issuance and validation
- Role-based access control (RBAC) definitions (admin, analyst, viewer)
- Session management and token refresh
- Shared read-only session view support

**Stories:** US-001, US-002, US-003

**Dependencies:** `infrastructure-and-platform`

---

## Unit 3 — Sandbox Environment

**Name:** `sandbox-environment`

**Boundaries:** Provides an always-on, isolated sandbox with anonymized/synthetic data for safe policy experimentation. Separate from production data.

**Responsibilities:**
- Dedicated DynamoDB tables for sandbox sessions and findings
- Synthetic/anonymized dataset generation pipeline
- Sandbox data refresh and reset mechanisms
- Isolation boundaries ensuring no production data leakage
- Sandbox configuration and environment variables management

**Stories:** US-012, US-013

**Dependencies:** `auth-and-identity`

---

## Unit 4 — Policy Repository

**Name:** `policy-repository`

**Boundaries:** Manages versioned credit policy files (SQL/Python scripts) stored in AWS CodeCommit. Provides APIs to list, browse, diff, and retrieve policy versions.

**Responsibilities:**
- AWS CodeCommit repository setup and IAM permissions
- Policy CRUD API (list, get, create branch, commit)
- Policy versioning and branch management
- Policy metadata storage in DynamoDB
- Diff and changelog endpoints for policy versions

**Stories:** US-004, US-005, US-006, US-007

**Dependencies:** `auth-and-identity`

---

## Unit 5 — Execution Engine

**Name:** `execution-engine`

**Boundaries:** Orchestrates on-demand execution of SQL/Python policy jobs as ECS Fargate tasks. Manages job lifecycle, status tracking, and result storage.

**Responsibilities:**
- ECS Fargate task definition and launch for SQL/Python jobs
- Job queue and execution orchestration
- Job status tracking in DynamoDB (pending, running, completed, failed)
- Polling API endpoint for frontend job status checks
- Execution logs capture and storage in S3
- Result output writing to S3 (Parquet/JSON)
- Sandbox environment routing (always-on dedicated environment)

**Stories:** US-008, US-009, US-010, US-011

**Dependencies:** `auth-and-identity`, `policy-repository`, `sandbox-environment`

---

## Unit 6 — Results & Analytics

**Name:** `results-and-analytics`

**Boundaries:** Handles storage, querying, and presentation of policy execution results. Uses Amazon Athena over S3 data lake for ad-hoc analytics.

**Responsibilities:**
- Athena workgroup and query configuration over S3 result data lake
- Results API (retrieve findings, hypotheses, execution outputs)
- DynamoDB storage for session findings and hypotheses
- Result comparison across policy versions
- Athena query result caching and pagination
- Audit trail append-only writes to S3 (Parquet/JSON)

**Stories:** US-014, US-015, US-016

**Dependencies:** `execution-engine`, `auth-and-identity`

---

## Unit 7 — Frontend Shell (Micro-Frontend Host)

**Name:** `frontend-shell`

**Boundaries:** The root SPA shell that bootstraps the micro-frontend architecture, handles global navigation, authentication flow, and lazy-loads feature SPAs.

**Responsibilities:**
- Micro-frontend host/shell setup (Module Federation or similar)
- Global navigation and routing
- Authentication flow (Cognito Hosted UI / redirect)
- Shared state and event bus for micro-frontends
- Theme, design system, and shared UI components
- Environment configuration injection

**Stories:** US-001, US-002, US-017

**Dependencies:** `auth-and-identity`

---

## Unit 8 — Policy Explorer Micro-Frontend

**Name:** `frontend-policy-explorer`

**Boundaries:** Independent SPA for browsing, diffing, and selecting policy versions from the repository. Communicates with the Policy Repository service.

**Responsibilities:**
- Policy list and search UI
- Policy version browser and diff viewer
- Branch and commit history display
- Policy selection for experiment setup
- Integration with Policy Repository API

**Stories:** US-004, US-005, US-006, US-007

**Dependencies:** `frontend-shell`, `policy-repository`

---

## Unit 9 — Experiment Runner Micro-Frontend

**Name:** `frontend-experiment-runner`

**Boundaries:** Independent SPA for configuring, launching, and monitoring policy execution jobs. Shows job status via polling and displays execution progress.

**Responsibilities:**
- Experiment configuration form (policy selection, parameters, sandbox target)
- Job submission to Execution Engine API
- Job status polling UI with progress indicators
- Execution log viewer
- Shared session support (read-only view for collaborators)

**Stories:** US-008, US-009, US-010, US-011, US-003

**Dependencies:** `frontend-shell`, `execution-engine`

---

## Unit 10 — Results Dashboard Micro-Frontend

**Name:** `frontend-results-dashboard`

**Boundaries:** Independent SPA for exploring execution results, findings, hypotheses, and cross-version comparisons through an Athena-backed interface.

**Responsibilities:**
- Results browsing and filtering UI
- Findings and hypotheses management
- Cross-version result comparison views
- Athena query builder / preset queries
- Export and reporting features
- Audit trail viewer

**Stories:** US-014, US-015, US-016

**Dependencies:** `frontend-shell`, `results-and-analytics`
