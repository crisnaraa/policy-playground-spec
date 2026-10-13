# Policy Playground — Spec Driven Development

> **Princípio:** AI proposes. Studies prove. Humans decide.

---

## REQUIREMENTS

### 1. Overview

**Product:** Policy Playground
**Domain:** Engenharia de Políticas de Crédito PF
**Version:** MVP v1.0
**Language:** Portuguese (BR) — interface e documentação

**Problem Statement:**
Especialistas de crédito possuem o conhecimento de negócio mas dependem de times técnicos para traduzir hipóteses em SQL/Python e executar estudos. Isso gera 3+ handoffs por hipótese e lead time médio de 3 dias. Além disso, a validação cobre apenas o que o especialista já sabe perguntar — combinações de regras, edge cases e comportamentos inesperados ficam de fora até aparecerem em produção.

**Solution Statement:**
O Policy Playground oferece uma experiência unificada onde o especialista expressa a intenção em linguagem natural, agentes de IA estruturam a hipótese e acionam os estudos existentes (fonte de verdade do cálculo), e um agente Red Team desafia proativamente a política gerando cenários não investigados. A decisão permanece sempre com o especialista.

---

### 2. User Personas

| Persona | Papel | Necessidade Principal |
|---|---|---|
| **Especialista de Políticas** (primary) | Formula hipóteses, propõe mudanças | Testar intenção sem código; entender consequências antes de decidir |
| **Analista de Dados** (secondary) | Executa e reproduz estudos | Menos pedidos avulsos; estudos reutilizados de forma governada |
| **Aprovador de Risco** (secondary) | Decide se a mudança segue | Evidência rastreável para aprovar ou rejeitar |
| **Engenharia do Motor** (secondary) | Implementa a política | Especificação sem ambiguidade |

---

### 3. Functional Requirements

#### FR-01 — Policy Viewer
The system shall allow the credit policy specialist to open any versioned credit policy stored in a Git repository and view its rules, variables, dependencies, and associated context in a readable, non-technical format.

**Acceptance Criteria:**
- [ ] Given a policy name, the system retrieves the latest versioned policy from Git
- [ ] The policy is displayed in a structured, human-readable format (not raw code)
- [ ] The Git commit/tag used is visible and recorded in the session

#### FR-02 — Natural Language Hypothesis Input
The system shall accept hypothesis descriptions in natural language (Portuguese), and convert them into structured hypotheses ready for execution.

**Acceptance Criteria:**
- [ ] The specialist can type a hypothesis (e.g., "Quero testar o que acontece se a renda mínima passar de R$2.500 para R$3.000")
- [ ] The AI agent returns a structured hypothesis with: affected rule(s), proposed change, and studies needed
- [ ] The structured hypothesis is presented for specialist review before execution

#### FR-03 — Visual Rule Editing
The system shall provide a visual interface where the specialist can modify a policy rule directly without writing code.

**Acceptance Criteria:**
- [ ] Rule parameters (thresholds, limits, flags) are editable via form controls
- [ ] Editing a rule automatically derives a structured hypothesis
- [ ] The visual editor reflects the current policy version from Git

#### FR-04 — Hypothesis Structuring Agent
An AI agent (Python-based) shall interpret the specialist's natural-language or visual-edit intent and produce a structured hypothesis.

**Acceptance Criteria:**
- [ ] The agent identifies which rules are affected
- [ ] The agent identifies which existing studies should be invoked
- [ ] The structured output is traceable back to the original intent
- [ ] Structuring completes within 30 seconds

#### FR-05 — Study Orchestration
The system shall programmatically invoke existing SQL/Python study jobs to execute the hypothesis experiment.

**Acceptance Criteria:**
- [ ] Studies are invoked via Step Functions state machine
- [ ] The Playground receives the study results without recomputing them
- [ ] Study execution is asynchronous; the UI shows progress in real time
- [ ] Results are cached in ElastiCache (Redis) for session reuse
- [ ] Persisted results stored in S3 and queryable via Athena

#### FR-06 — Red Team Agent (Policy Challenger)
Concurrently with the hypothesis test, an AI agent (Python-based) shall proactively analyze the policy and generate additional investigative scenarios.

**Acceptance Criteria:**
- [ ] The Red Team agent runs in parallel with study execution
- [ ] It generates findings for: rule conflicts, edge cases, unexpected segment behaviors, and uncovered gaps
- [ ] Findings are grouped by type and ordered by estimated impact
- [ ] Findings are published to SQS; the findings-service consumes them asynchronously
- [ ] Number of findings per session is capped (configurable)

#### FR-07 — Findings Presentation with Traceability
The system shall present all AI-generated findings with full traceability.

**Acceptance Criteria:**
- [ ] Each finding shows: the rule(s) involved, the data used, and the study that supports it
- [ ] Findings without deterministic evidence are labeled "hipótese não comprovada"
- [ ] The specialist can expand each finding to see the supporting evidence

#### FR-08 — Finding Relevance Feedback
The specialist shall be able to mark each finding as "relevant" or "irrelevant".

**Acceptance Criteria:**
- [ ] One-click relevance marking (relevant / irrelevant)
- [ ] Feedback is persisted to DynamoDB
- [ ] Feedback retroactively re-prioritizes remaining findings in the session

#### FR-09 — Finding-to-Scenario Conversion
The specialist shall be able to convert any AI finding into a new testable scenario.

**Acceptance Criteria:**
- [ ] A "Convert to Scenario" action is available on each finding
- [ ] Converting creates a new hypothesis pre-filled with the finding's context
- [ ] The new scenario can be explored in the same session

#### FR-10 — Hypothesis Decision Recording
After reviewing results, the specialist shall explicitly record their decision on the hypothesis.

**Acceptance Criteria:**
- [ ] Available decisions: Accept / Reject / Defer
- [ ] Decision is persisted with timestamp, specialist identity, and associated evidence
- [ ] Decision is immutable once recorded (requires admin override to change)

#### FR-11 — Session State Management
The system shall persist the full state of each session.

**Acceptance Criteria:**
- [ ] Session state includes: policy version, hypothesis text, structured hypothesis, studies invoked, results, findings, evaluations, and decision
- [ ] Sessions can be resumed after interruption
- [ ] Sessions are associated with the authenticated user

#### FR-12 — Findings Prioritization and Session Limit
Findings shall be prioritized by impact and limited per session.

**Acceptance Criteria:**
- [ ] Findings are sorted by estimated impact (high → low)
- [ ] A configurable maximum number of findings per session (default: 10)
- [ ] Findings beyond the limit are queued and surfaced on specialist request

#### FR-13 — Policy Version Tracking
The system shall record which Git version of the policy was used in each session.

**Acceptance Criteria:**
- [ ] Git commit hash / tag is recorded at session creation
- [ ] Sessions are reproducible using the recorded version

#### FR-14 — Immutable Audit Trail
Every action (hypothesis, study execution, finding, feedback, decision) shall be recorded immutably.

**Acceptance Criteria:**
- [ ] Audit records are written to Amazon S3 in Parquet/JSON format
- [ ] S3 Object Lock (WORM) is enabled on the audit bucket
- [ ] Records include: timestamp, user ID, action type, and full payload
- [ ] Audit records are queryable via Amazon Athena

#### FR-15 — Corporate Identity Authentication
The system shall authenticate users via corporate identity federation.

**Acceptance Criteria:**
- [ ] Users log in via SAML/OIDC connected to the corporate Active Directory
- [ ] Authentication is handled by Amazon Cognito with federated identity
- [ ] Unauthenticated access is denied

#### FR-16 — Role-Based Access Control (RBAC)
The system shall enforce role-based permissions.

**Acceptance Criteria:**
- [ ] Roles: Policy Specialist, Risk Approver, Data Analyst, Admin
- [ ] Each role has a defined permission set enforced at API level
- [ ] Role assignments are managed by Admin users

#### FR-17 — LGPD-Compliant Data Handling
Experiments shall use anonymized or synthetic data. No PII shall be exposed.

**Acceptance Criteria:**
- [ ] PII is not displayed in the experiment interface, findings, or AI prompts
- [ ] Data used in experiments comes from the governed anonymized/synthetic dataset
- [ ] Data access is logged and role-restricted

#### FR-18 — FinOps: Cost Control per Session
The system shall enforce cost controls on AI inference and study executions.

**Acceptance Criteria:**
- [ ] Per-session limit on Bedrock inference calls (configurable, tracked in DynamoDB)
- [ ] Study results are cached in ElastiCache (Redis) and reused within the session
- [ ] Admin dashboard shows cost per hypothesis and cumulative session costs via CloudWatch

---

### 4. Non-Functional Requirements

| ID | Category | Requirement | Target |
|---|---|---|---|
| NFR-01 | Cloud | AWS-only infrastructure | 100% AWS |
| NFR-02 | Deployment | All services containerized on Amazon ECS Fargate (multi-AZ) | ≥ 2 AZs |
| NFR-03 | Frontend | React/Next.js on S3 + CloudFront | < 2s TTI |
| NFR-04 | AI | Amazon Bedrock (Claude) for all LLM calls | Bedrock API |
| NFR-05 | Database | Amazon DynamoDB for session/state/findings storage | On-demand capacity |
| NFR-06 | Cache | Amazon ElastiCache (Redis) for study result caching | Sub-millisecond reads |
| NFR-07 | Analytics | Amazon Athena for experiment result queries | S3 as source |
| NFR-08 | Audit | S3 Object Lock (WORM) for audit records | Append-only, immutable |
| NFR-09 | Messaging | Amazon SQS for async findings delivery (agent → findings-service) | At-least-once delivery |
| NFR-10 | Integration | Read-only Git integration for policy rules | No write access |
| NFR-11 | Integration | Study orchestration via AWS Step Functions | Async execution |
| NFR-12 | Performance | Structured hypothesis returned within 30s | P95 ≤ 30s |
| NFR-13 | Performance | Study execution async with real-time updates | WebSocket / polling |
| NFR-14 | Compliance | LGPD: no PII in LLM prompts or interface | Zero PII exposure |
| NFR-15 | Availability | 99.9% uptime | SLA |
| NFR-16 | Scalability | Architecture supports future scale | ECS auto-scale, DynamoDB on-demand |
| NFR-17 | Observability | AWS Distro for OpenTelemetry (ADOT) + CloudWatch + X-Ray | Full distributed tracing |
| NFR-18 | IaC | All infrastructure defined in AWS CDK (TypeScript) | 100% IaC, no manual resources |
| NFR-19 | Security | TLS 1.2+, KMS encryption, Secrets Manager, IAM least-privilege | All data at rest + transit |

---

### 5. Out of Scope (MVP v1.0)

- [ ] Code, test, and Pull Request generation (future evolution)
- [ ] Autonomous changes to production policies
- [ ] Creation of new studies (MVP uses only existing studies)
- [ ] Definition of which specific policies/products are in scope (business decision pending)

---

### 6. Success Metrics

| Metric | Baseline | Target |
|---|---|---|
| Lead time: hypothesis → evidence | 3 days | 1 day |
| Handoffs per hypothesis | 3 | 1 |
| % hypotheses tested without technical support | 25% | 75% |
| % AI findings rated as relevant | — | 55% |
| Unexpected behaviors detected before production | 33% | 60% |

---

## DESIGN

### 1. Architecture Overview

The Policy Playground uses a **hybrid backend architecture**:
- **Go** services handle high-frequency, low-latency API paths (session management, policy retrieval, findings CRUD, auth)
- **Python** services handle the AI/agent layer (Bedrock integration, hypothesis structuring, Red Team agent, study orchestration)
- **Amazon SQS** decouples the Python agent from the Go findings-service for async finding delivery
- **Amazon ElastiCache (Redis)** provides distributed caching of study results across all Fargate instances
- **AWS CDK (TypeScript)** defines all infrastructure as code

```
┌──────────────────────────────────────────────────────────────┐
│                     CloudFront CDN                           │
│                 (React/Next.js SPA on S3)                    │
└────────────────────────┬─────────────────────────────────────┘
                         │ HTTPS
┌────────────────────────▼─────────────────────────────────────┐
│                 Amazon API Gateway                           │
│             (REST API + WebSocket for async)                 │
└──────────────┬─────────────────────────┬─────────────────────┘
               │                         │
┌──────────────▼───────────┐  ┌──────────▼───────────────────┐
│    Go API Services       │  │  Python AI Agent Services    │
│    (ECS Fargate)         │  │  (ECS Fargate)               │
│                          │  │                              │
│ • session-service        │  │ • agent-service (Bedrock)    │
│ • policy-service (Git)   │  │ • study-orchestrator         │
│ • findings-service ◄─SQS─┼──┤   (Step Functions)          │
│ • decision-service       │  │                              │
└──────────┬───────────────┘  └──────────┬───────────────────┘
           │                             │
    ┌──────▼─────────┐         ┌─────────▼──────────┐
    │   DynamoDB     │         │  Amazon Bedrock     │
    │ (sessions,     │         │  (Claude LLM +      │
    │  findings,     │         │   Bedrock Agents)   │
    │  decisions)    │         └─────────┬──────────┘
    └────────────────┘                   │
                              ┌──────────▼──────────┐
    ┌───────────────┐         │   Step Functions    │
    │  ElastiCache  │◄────────┤  (Study Executor)   │
    │  Redis        │ cache   └──────────┬──────────┘
    │  (results)    │                    │
    └───────────────┘         ┌──────────▼──────────┐
                              │  Existing Studies   │
                              │ (SQL/Python Jobs)   │
                              └──────────┬──────────┘
                                         │
                    ┌────────────────────▼──────────────┐
                    │  S3 Results + Athena               │
                    │  S3 Audit Bucket (WORM/Object Lock)│
                    └────────────────────────────────────┘

All infrastructure provisioned via AWS CDK (TypeScript)
Observability: AWS Distro for OpenTelemetry (ADOT) → CloudWatch + X-Ray
```

### 2. Service Decomposition

| Service | Language | Responsibility |
|---|---|---|
| **session-service** | Go | Create/read/update sessions; session state in DynamoDB |
| **policy-service** | Go | Read policies from Git; parse and serve rule structure |
| **findings-service** | Go | CRUD for findings; consumes SQS; relevance feedback; re-ranking |
| **decision-service** | Go | Record and retrieve hypothesis decisions (immutable) |
| **agent-service** | Python | Hypothesis structuring + Red Team agent via Bedrock; publishes findings to SQS |
| **study-orchestrator** | Python | Invoke Step Functions; manage ElastiCache result cache; relay results |

### 3. AWS Services

| Layer | Service | Purpose |
|---|---|---|
| Frontend | S3 + CloudFront | Static hosting + CDN |
| API | Amazon API Gateway | REST + WebSocket |
| Go Backend | Amazon ECS Fargate | High-frequency CRUD services |
| Python AI Backend | Amazon ECS Fargate | AI agent + study orchestration |
| AI/Agent | Amazon Bedrock (Claude) + Bedrock Agents | LLM inference + agentic orchestration |
| Async Messaging | Amazon SQS | Decoupled findings delivery (Python → Go) |
| State/Session | Amazon DynamoDB | Session state, findings, decisions |
| Cache | Amazon ElastiCache (Redis) | Study result caching across Fargate instances |
| Study Results | Amazon S3 + Athena | Experiment outputs + ad hoc queries |
| Audit | Amazon S3 (Object Lock) | Immutable append-only audit data lake |
| Auth | Amazon Cognito (SAML federation) | Corporate AD integration |
| Study Orchestration | AWS Step Functions | Async job execution |
| Container Registry | Amazon ECR | Docker images (Go and Python) |
| Secrets | AWS Secrets Manager | API keys, Git tokens, credentials |
| Observability | ADOT + CloudWatch + X-Ray | Unified distributed tracing across Go + Python |
| IaC | AWS CDK (TypeScript) | All infrastructure as code |
| Policy Source | Git (read-only integration) | Policy rule versioning |

### 4. Data Model (DynamoDB)

**Table: Sessions**
```
PK: session_id (UUID)
SK: "SESSION"
Attributes: user_id, policy_id, policy_git_ref, hypothesis_text,
            structured_hypothesis, status, created_at, updated_at
```

**Table: Findings**
```
PK: session_id
SK: finding_id (UUID)
Attributes: type, description, affected_rules, supporting_study,
            evidence_s3_key, relevance_feedback, impact_score, created_at
```

**Table: Decisions**
```
PK: session_id
SK: "DECISION"
Attributes: decision (accept|reject|defer), decided_by, decided_at,
            rationale, hypothesis_ref
```

### 5. Async Messaging Flow (SQS)

```
Python agent-service
  → publishes FindingEvent to SQS queue (findings-queue)
  → message: { session_id, finding_id, type, description, rules, evidence_s3_key, impact_score }

Go findings-service
  → polls SQS (long polling, batch=10)
  → writes finding to DynamoDB
  → pushes WebSocket notification to API Gateway (session subscribers)
  → acknowledges SQS message (delete)
```

### 6. Cache Strategy (ElastiCache Redis)

```
Key: study_result:{study_id}:{parameters_hash}
TTL: 1 hour (configurable)
Value: S3 key pointing to result Parquet file

Cache hit: Go study-orchestrator returns cached result immediately
Cache miss: triggers Step Functions execution → stores result → caches S3 key
```

### 7. Observability (ADOT)

- AWS Distro for OpenTelemetry (ADOT) collector sidecar on all ECS tasks
- Traces propagated via W3C TraceContext headers across Go ↔ Python ↔ Bedrock ↔ Step Functions
- Metrics: custom CloudWatch metrics for hypothesis_structured, study_executed, finding_published, decision_recorded
- Dashboards: session volume, P95 latency, Bedrock token consumption, SQS queue depth, Redis hit rate

### 8. Security Design

- **Authentication:** Amazon Cognito User Pool with SAML identity provider (corporate AD)
- **Authorization:** JWT tokens with custom role claims; enforced at API Gateway + each service
- **Data Encryption:** S3 (SSE-KMS), DynamoDB (AWS-managed KMS), ElastiCache (encryption at rest + in transit), secrets via AWS Secrets Manager
- **Network:** VPC with private subnets for ECS tasks; public-facing only via API Gateway + CloudFront
- **LGPD:** PII stripped at data ingestion; anonymized/synthetic data only; no PII in Bedrock prompts

### 9. Infrastructure as Code (AWS CDK)

All AWS resources are provisioned via **AWS CDK (TypeScript)** organized in stacks:

| CDK Stack | Resources |
|---|---|
| `NetworkStack` | VPC, subnets, security groups, NAT Gateways |
| `AuthStack` | Cognito User Pool, SAML IdP, Cognito App Client |
| `DatabaseStack` | DynamoDB tables, ElastiCache Redis cluster |
| `StorageStack` | S3 buckets (results, audit with Object Lock), ECR repos |
| `MessagingStack` | SQS queues (findings-queue, DLQ) |
| `GoServicesStack` | ECS Cluster, Fargate services for Go tier, ALB |
| `PythonServicesStack` | ECS Fargate services for Python AI tier, ALB |
| `ObservabilityStack` | CloudWatch dashboards, alarms, X-Ray groups, ADOT config |
| `FrontendStack` | S3 bucket, CloudFront distribution, ACM certificate |
| `OrchestrationStack` | Step Functions state machine, Lambda functions |

---

## TASKS

### Phase 1: Foundation (Sprint 1–2)

#### TASK-001 — CDK Project & AWS Infrastructure
- [ ] Initialize AWS CDK TypeScript project (`cdk init app --language typescript`)
- [ ] Implement `NetworkStack`: VPC, 2 AZs, public/private subnets, NAT Gateways
- [ ] Implement `AuthStack`: Cognito User Pool + SAML federation with corporate AD
- [ ] Implement `DatabaseStack`: DynamoDB tables (Sessions, Findings, Decisions) + ElastiCache Redis cluster
- [ ] Implement `StorageStack`: S3 buckets (results, audit with Object Lock enabled), ECR repos
- [ ] Implement `MessagingStack`: SQS findings-queue + DLQ
- [ ] `cdk deploy --all` to dev environment

#### TASK-002 — Go Services Skeleton
- [ ] Scaffold Go monorepo (one module per service: session, policy, findings, decision)
- [ ] Implement health check endpoints (`GET /health`) for all Go services
- [ ] Implement JWT validation middleware (Cognito public keys via JWKS)
- [ ] Implement RBAC middleware with role claim enforcement
- [ ] Instrument with ADOT (OpenTelemetry Go SDK + ADOT sidecar)
- [ ] Dockerize Go services and push to ECR
- [ ] Implement `GoServicesStack` CDK: ECS Cluster, Fargate services, internal ALB

#### TASK-003 — Python Agent Service Skeleton
- [ ] Scaffold Python FastAPI project (agent-service + study-orchestrator)
- [ ] Configure AWS Boto3 + Amazon Bedrock SDK
- [ ] Implement health check endpoint
- [ ] Instrument with ADOT (OpenTelemetry Python SDK + ADOT sidecar)
- [ ] Dockerize Python services and push to ECR
- [ ] Implement `PythonServicesStack` CDK: ECS Fargate services, internal ALB

#### TASK-004 — Frontend Skeleton
- [ ] Scaffold React/Next.js project
- [ ] Implement `FrontendStack` CDK: S3 bucket + CloudFront distribution
- [ ] Implement authentication UI with Amazon Cognito Hosted UI
- [ ] Set up routing: `/policies`, `/session/:id`, `/findings`
- [ ] Connect to API Gateway endpoints (dev environment)

---

### Phase 2: Core Features (Sprint 3–5)

#### TASK-005 — Policy Service (Go)
- [ ] Implement Git read-only integration (token via Secrets Manager)
- [ ] `GET /policies` — list available policies from Git repo
- [ ] `GET /policies/:id` — retrieve policy with parsed rules (YAML/JSON → structured)
- [ ] Policy displayed in human-readable format on frontend
- [ ] Git ref recorded in session at creation

#### TASK-006 — Session Service (Go)
- [ ] `POST /sessions` — create session (policy_id, hypothesis_text or visual_edit)
- [ ] `GET /sessions/:id` — retrieve full session state from DynamoDB
- [ ] `PATCH /sessions/:id` — update session status
- [ ] Session includes Git ref; stored in DynamoDB

#### TASK-007 — Hypothesis Structuring Agent (Python + Bedrock)
- [ ] Configure Amazon Bedrock Claude model invocation
- [ ] Implement Bedrock Agents tool definitions for hypothesis structuring
- [ ] Agent prompt: parse natural language → affected rules → required studies
- [ ] `POST /agent/structure` — accepts hypothesis + policy context, returns structured hypothesis
- [ ] Call from Go session-service to Python agent-service via internal ALB (HTTP)
- [ ] Define OpenAPI contract between Go and Python (TASK-007-CONTRACT)
- [ ] Structured hypothesis stored in DynamoDB session
- [ ] P95 response time ≤ 30s validated in load test

#### TASK-008 — Study Orchestrator (Python + Step Functions + Redis)
- [ ] Implement `OrchestrationStack` CDK: Step Functions state machine + Lambda invocators
- [ ] `POST /agent/execute` — checks Redis cache first; cache miss triggers Step Functions
- [ ] Store study results in S3; cache S3 key in Redis with 1h TTL
- [ ] `GET /sessions/:id/status` WebSocket relay via API Gateway + Lambda
- [ ] Study result S3 key persisted in DynamoDB session

#### TASK-009 — Results Display (Frontend)
- [ ] Results panel: impact metrics, affected segments, study name
- [ ] Athena query integration for ad hoc result exploration
- [ ] Display policy Git ref and study execution timestamp

---

### Phase 3: AI Challenge & Findings (Sprint 6–7)

#### TASK-010 — Red Team Agent (Python + Bedrock + SQS)
- [ ] Implement Red Team agent using Amazon Bedrock Agents
- [ ] Agent tools: analyze rule graph, identify edge cases, detect conflicts, suggest gaps
- [ ] Run in parallel with study execution (Step Functions parallel branch)
- [ ] Publish finding events to SQS findings-queue (one message per finding)
- [ ] Configurable finding cap per session (DynamoDB counter)

#### TASK-011 — Findings Service (Go + SQS Consumer)
- [ ] Implement SQS long-poll consumer in findings-service (goroutine pool)
- [ ] `GET /sessions/:id/findings` — list findings (sorted by impact, grouped by type)
- [ ] `PATCH /findings/:id/feedback` — record relevance feedback; update impact rank
- [ ] `POST /findings/:id/convert` — convert finding to new scenario hypothesis
- [ ] Push WebSocket notification to API Gateway on new finding received
- [ ] DLQ monitoring alarm in `ObservabilityStack` CDK

#### TASK-012 — Findings UI (Frontend)
- [ ] Findings panel with grouped/sorted finding cards (WebSocket live updates)
- [ ] Each card: type badge, description, affected rules, evidence link, relevance buttons
- [ ] Findings without evidence labeled "hipótese não comprovada"
- [ ] "Convert to Scenario" → pre-populates hypothesis form

---

### Phase 4: Decision & Governance (Sprint 8)

#### TASK-013 — Decision Service (Go)
- [ ] `POST /sessions/:id/decision` — record decision (accept/reject/defer), immutable
- [ ] `GET /sessions/:id/decision` — retrieve decision with full evidence context
- [ ] Frontend: decision panel with summary modal (hypothesis + results + findings reviewed)
- [ ] Audit event emitted on decision recorded

#### TASK-014 — Audit Trail Implementation
- [ ] Audit event publisher: async background goroutine (Go) + background task (Python)
- [ ] Events published: session_created, hypothesis_structured, study_executed, finding_generated, finding_evaluated, decision_recorded
- [ ] Write events to S3 audit bucket (Parquet/JSON, partitioned by `year/month/day`)
- [ ] Verify S3 Object Lock WORM enforcement
- [ ] Athena table + Glue catalog for audit queries
- [ ] Implement `ObservabilityStack` CDK: CloudWatch dashboards, alarms

---

### Phase 5: Hardening & FinOps (Sprint 9)

#### TASK-015 — LGPD & Data Governance
- [ ] Verify no PII in DynamoDB records, SQS messages, or Redis cache values
- [ ] Confirm anonymized/synthetic dataset used in all study executions
- [ ] Implement data access logging (DynamoDB Streams → CloudWatch)
- [ ] Privacy impact documentation for LGPD compliance review

#### TASK-016 — FinOps Controls
- [ ] Per-session Bedrock inference call counter (DynamoDB atomic increment in agent-service)
- [ ] Configurable limit; return 429 when exceeded
- [ ] Redis cache hit rate metric (custom CloudWatch metric)
- [ ] CloudWatch dashboard: Bedrock token consumption, cost per session, SQS depth, Redis hit/miss

#### TASK-017 — Observability Hardening
- [ ] ADOT sidecar configured on all ECS tasks (Go and Python)
- [ ] W3C TraceContext propagation validated across Go ↔ Python ↔ Bedrock ↔ Step Functions
- [ ] CloudWatch Log Insights queries for: error rate, P95 latency, finding volume, decision rate
- [ ] X-Ray service map covering full request path
- [ ] CloudWatch Alarms: error rate > 1%, P95 > 30s, DynamoDB throttle, SQS DLQ depth > 0

---

### Phase 6: Launch Readiness (Sprint 10)

#### TASK-018 — End-to-End Testing
- [ ] Integration tests: full hypothesis flow across Go + Python (session → structure → execute → findings → decision)
- [ ] Load test: 20 concurrent sessions; validate Go tier handles high-frequency CRUD without bottleneck
- [ ] SQS consumer backpressure test: high finding volume without data loss
- [ ] Security test: RBAC enforcement, JWT expiry, injection attempts
- [ ] LGPD test: zero PII in any API response, SQS message, Redis value, or Bedrock prompt

#### TASK-019 — Documentation
- [ ] OpenAPI/Swagger specs for all Go and Python services
- [ ] User guide for Policy Specialists (Portuguese)
- [ ] Admin guide: role management, FinOps controls, audit queries, CDK deployment
- [ ] Architecture Decision Records (ADRs):
  - ADR-001: Go + Python hybrid rationale
  - ADR-002: SQS for findings decoupling
  - ADR-003: ElastiCache Redis for study result caching
  - ADR-004: AWS CDK TypeScript for IaC

#### TASK-020 — Go-Live Checklist
- [ ] AWS WAF configured on CloudFront and API Gateway (OWASP Top 10 rules)
- [ ] DynamoDB point-in-time recovery enabled; S3 versioning on results bucket
- [ ] ElastiCache Redis multi-AZ with automatic failover enabled
- [ ] `cdk deploy --all` to production environment from CI/CD pipeline
- [ ] Disaster recovery playbook documented
- [ ] Stakeholder demo and sign-off

---

## DEPENDENCIES

| Dependency | Owner | Required By |
|---|---|---|
| Programmatic access to existing SQL/Python studies | Data Engineering | TASK-008 |
| Governed anonymized/synthetic dataset | Data Governance | TASK-008 |
| Corporate SAML IdP metadata for Cognito | IT/IAM | TASK-001 |
| Git repository read access token | DevOps | TASK-005 |
| Definition of which policies are in v1 scope | Business/Credit Team | TASK-005 |
| Amazon Bedrock Claude model access enabled in AWS account | Cloud/Platform | TASK-007 |
