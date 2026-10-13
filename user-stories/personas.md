# User Personas — Policy Playground

## Overview
Policy Playground serves four primary personas, each with distinct goals
and interaction patterns within the platform.

---

## Persona 1 — Credit Analyst

**Name:** Ana Rodrigues
**Role:** Senior Credit Analyst
**Team:** Credit Risk

### Background
Ana has 7 years of experience in credit risk modeling. She understands business
logic deeply but relies on data scientists for complex SQL/Python work. She needs
a way to test policy ideas quickly without waiting for IT deployments.

### Goals
- Compare different policy versions side-by-side
- Create "what if" hypotheses without touching production
- Review findings from data science team and make recommendations

### Pain Points
- Policy changes take weeks to deploy in the current process
- No easy way to track which experiment led to which business decision
- Results are scattered across emails and spreadsheets

### Platform Usage
- Creates and manages hypotheses
- Reviews job results via the UI (tables/charts)
- Accesses audit logs to trace past decisions

---

## Persona 2 — Data Scientist

**Name:** Carlos Mendes
**Role:** Data Scientist
**Team:** Analytics & Modeling

### Background
Carlos builds and maintains SQL and Python scripts that evaluate credit policy
performance. He is comfortable with code but wants a streamlined UI to trigger
runs, track status, and share results with analysts.

### Goals
- Run experiments on synthetic data without manual setup
- Track job status and access results quickly
- Upload and manage synthetic datasets for testing

### Pain Points
- Manually provisioning infrastructure for each experiment is time-consuming
- No single place to see all experiment history
- Results sharing with analysts is cumbersome

### Platform Usage
- Triggers on-demand SQL/Python jobs
- Manages synthetic datasets
- Links job results to hypotheses

---

## Persona 3 — Risk Manager

**Name:** Fernanda Lima
**Role:** Head of Credit Risk
**Team:** Risk Management

### Background
Fernanda oversees all credit policy decisions. She needs visibility into what
experiments are being run, what results look like, and whether changes are
compliant before they are promoted to production.

### Goals
- Review experiment findings at a high level
- Ensure all changes are traceable and auditable
- Approve or reject policy hypotheses before production

### Pain Points
- Difficult to audit who made what decision and when
- No structured approval workflow before policy goes live

### Platform Usage
- Views experiment summaries and findings
- Accesses the immutable audit log
- Reviews hypothesis lists and linked results

---

## Persona 4 — Platform Admin

**Name:** Rafael Costa
**Role:** Platform Engineer / DevOps
**Team:** Infrastructure

### Background
Rafael is responsible for maintaining the Policy Playground platform on AWS.
He manages user access, monitors infrastructure, and ensures the platform
is secure, reliable, and scalable.

### Goals
- Manage user roles and permissions centrally
- Monitor platform health and job queue
- Ensure compliance with data governance policies

### Pain Points
- User onboarding via Azure AD requires manual role assignment
- No centralized view of platform activity

### Platform Usage
- Assigns roles to new users
- Monitors system health
- Reviews audit logs for anomalies
