# User Stories — Policy Playground

## Overview
Policy Playground is a web-based platform that allows credit analysts and data scientists
to simulate, version, and validate credit policies using SQL/Python jobs, synthetic data,
and Git-versioned policy definitions. All infrastructure runs on AWS (ECS Fargate / EKS),
with DynamoDB, Athena, and S3 as the data backbone. Auth is via Azure AD federation.

---

## Epic 1 — Authentication & Access

### US-01: Corporate Login
**As a** corporate user,
**I want to** log in using my company Azure AD credentials,
**So that** I don't need to manage a separate username/password.

**Acceptance Criteria:**
- [ ] User is redirected to Azure AD login page when accessing the platform
- [ ] Successful login grants access to the dashboard
- [ ] Failed/unauthorized login shows a clear error message
- [ ] Session expires after inactivity (configurable, default 8h)
- [ ] Logout clears session and redirects to login page

**INVEST:** Independent ✅ | Negotiable ✅ | Valuable ✅ | Estimable ✅ | Small ✅ | Testable ✅

---

## Epic 2 — Policy Management

### US-02: View Policy Versions
**As a** credit analyst,
**I want to** browse all versions of a credit policy from the Git repository,
**So that** I can understand how the policy has evolved over time.

**Acceptance Criteria:**
- [ ] Platform lists all available policy versions with commit hash, date, and author
- [ ] User can view the content/diff of any version
- [ ] User can select a specific version to use in a simulation

**INVEST:** Independent ✅ | Negotiable ✅ | Valuable ✅ | Estimable ✅ | Small ✅ | Testable ✅

---

### US-03: Create / Edit Policy Hypothesis
**As a** credit analyst,
**I want to** create a named hypothesis that combines a policy version with parameter overrides,
**So that** I can test "what if" scenarios without modifying the canonical policy.

**Acceptance Criteria:**
- [ ] User can create a hypothesis with a name, description, base policy version, and parameter overrides
- [ ] Hypothesis is saved in DynamoDB and tagged with creator and timestamp
- [ ] User can edit or delete their own hypotheses
- [ ] Hypotheses are listed on a dedicated page with filters (status, author, date)

**INVEST:** Independent ✅ | Negotiable ✅ | Valuable ✅ | Estimable ✅ | Small ✅ | Testable ✅

---

## Epic 3 — Job Execution (SQL / Python)

### US-04: Execute an On-Demand Job
**As a** data scientist,
**I want to** trigger a SQL or Python job against synthetic/anonymized data,
**So that** I can validate how a credit policy performs on realistic data.

**Acceptance Criteria:**
- [ ] User can select a job (SQL or Python script) from a catalogue
- [ ] User can select the target dataset (synthetic/anonymized)
- [ ] Job is submitted and queued; user sees a status indicator (queued → running → completed/failed)
- [ ] On completion, user is notified and results are accessible

**INVEST:** Independent ✅ | Negotiable ✅ | Valuable ✅ | Estimable ✅ | Small ✅ | Testable ✅

---

### US-05: View Job Results
**As a** data scientist,
**I want to** view the results of a completed job in a readable format,
**So that** I can interpret findings and decide on next steps.

**Acceptance Criteria:**
- [ ] Results are displayed as a table or chart (based on output type)
- [ ] Raw output is available for download (CSV/Parquet)
- [ ] Athena is used to query result data for large outputs
- [ ] Results are linked to the hypothesis and job that produced them

**INVEST:** Independent ✅ | Negotiable ✅ | Valuable ✅ | Estimable ✅ | Small ✅ | Testable ✅

---

## Epic 4 — Audit & Traceability

### US-06: Audit Log Access
**As a** risk manager,
**I want to** view an immutable audit log of all experiments and policy changes,
**So that** I can ensure compliance and traceability.

**Acceptance Criteria:**
- [ ] All job executions, hypothesis creations, and policy selections are logged to S3 (append-only, Parquet/JSON)
- [ ] Audit log is queryable via the platform UI (date range, user, action type)
- [ ] Log entries cannot be deleted or modified
- [ ] Export of audit log to CSV is available

**INVEST:** Independent ✅ | Negotiable ✅ | Valuable ✅ | Estimable ✅ | Small ✅ | Testable ✅

---

## Epic 5 — Data Management

### US-07: Manage Synthetic Datasets
**As a** data scientist,
**I want to** upload or generate synthetic/anonymized datasets derived from real data,
**So that** experiments are realistic but privacy-safe.

**Acceptance Criteria:**
- [ ] User can upload a synthetic dataset (CSV/Parquet) to S3 via the platform
- [ ] Platform validates schema on upload and rejects invalid files
- [ ] Datasets are listed with name, size, upload date, and creator
- [ ] Datasets can be marked as active/inactive for job targeting

**INVEST:** Independent ✅ | Negotiable ✅ | Valuable ✅ | Estimable ✅ | Small ✅ | Testable ✅

---

## Epic 6 — Platform Administration

### US-08: Manage Users & Roles
**As a** platform admin,
**I want to** assign roles (Analyst, Scientist, Manager, Admin) to authenticated users,
**So that** access control is enforced throughout the platform.

**Acceptance Criteria:**
- [ ] Admin can list all users who have logged in via Azure AD
- [ ] Admin can assign or revoke roles
- [ ] Role changes take effect on next login or session refresh
- [ ] Role-based UI elements are shown/hidden based on current user's role

**INVEST:** Independent ✅ | Negotiable ✅ | Valuable ✅ | Estimable ✅ | Small ✅ | Testable ✅

---

## Story Summary

| Story ID | Title | Epic | Priority |
|---|---|---|---|
| US-01 | Corporate Login | Auth | Must Have |
| US-02 | View Policy Versions | Policy Management | Must Have |
| US-03 | Create / Edit Policy Hypothesis | Policy Management | Must Have |
| US-04 | Execute On-Demand Job | Job Execution | Must Have |
| US-05 | View Job Results | Job Execution | Must Have |
| US-06 | Audit Log Access | Audit & Traceability | Must Have |
| US-07 | Manage Synthetic Datasets | Data Management | Should Have |
| US-08 | Manage Users & Roles | Platform Admin | Must Have |
