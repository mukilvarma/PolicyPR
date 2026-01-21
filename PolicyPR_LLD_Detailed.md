# PolicyPR — Low Level Design (LLD)

Version: 1.0  
Last Updated: 2026-01-21  
Target Platform: Azure  
Integration: GitHub App  
Primary Goals: Scalability, Extensibility, Privacy-first AI-assisted PR reviews

---

## 1. Purpose of this Document

This Low Level Design (LLD) document describes the **internal architecture, components, data models, and execution flow** of **PolicyPR**.

It is intended for:
- Software architects
- Backend / platform engineers
- Security & compliance reviewers
- Contributors extending the system

This document goes beyond a high-level overview and explains **how PolicyPR works internally**.

---

## 2. System Overview

PolicyPR is an **event-driven, serverless pull request review platform**.

It integrates with GitHub via a **GitHub App**, analyzes pull requests using **deterministic tooling**, applies **organization-defined policies**, optionally enriches output using **AI (privacy-controlled)**, and publishes results back to GitHub as **Check Runs and PR comments**.

### Key Principles
- Deterministic analysis is the source of truth
- AI is optional and explainability-focused
- Policies control behavior, scoring, and data flow
- No source code leaves the environment by default

---

## 3. Functional Requirements

### 3.1 Core Capabilities
- Automatically trigger on PR open/update
- Fetch PR metadata, files, and diffs securely
- Detect security, quality, and compliance issues
- Identify high-risk change hotspots
- Compute quality scores and enforce gates
- Publish structured feedback inside GitHub

### 3.2 Supported Review Dimensions
- Security
- Privacy & Compliance (HIPAA / GDPR)
- Reliability
- Performance
- Maintainability
- Test Coverage
- Observability

---

## 4. Non-Functional Requirements (NFR)

### 4.1 Scalability
- Handle burst PR traffic across orgs
- Queue-based decoupling using Azure Service Bus
- Parallel analysis with bounded concurrency
- Stateless compute via Azure Functions

### 4.2 Reliability
- At-least-once processing
- Idempotency based on PR + commit SHA
- Durable orchestration with retry semantics
- Dead Letter Queue for poison jobs

### 4.3 Security & Compliance
- GitHub webhook signature verification
- Short-lived GitHub App installation tokens
- Secrets managed via Azure Key Vault
- Privacy modes to control AI usage
- Full audit trail for analysis steps

### 4.4 Observability
- Correlation IDs per review job
- Centralized logging via Application Insights
- Metrics: latency, queue depth, failure rate
- Alerts on DLQ growth and repeated failures

---

## 5. High-Level Architecture

### Azure Components
- Azure Functions (HTTP Trigger)
- Azure Service Bus (Queue + DLQ)
- Azure Durable Functions (Orchestration)
- Azure Cosmos DB
- Azure Blob Storage
- Azure AI Search (Vector Store for RAG)
- Azure Key Vault
- Application Insights

### External Systems
- GitHub (PR events, API calls)
- Optional AI Providers (Gemini / Azure OpenAI)

---

## 6. GitHub Integration Design

### 6.1 GitHub App
- Installed at org or repo level
- Receives PR webhooks
- Uses installation tokens for API calls

### 6.2 Required Permissions
- Read: Pull requests, repository contents, metadata
- Write: Checks, pull request comments

### 6.3 Events Consumed
- pull_request.opened
- pull_request.synchronize
- pull_request.reopened
- pull_request.ready_for_review

---

## 7. Detailed Processing Flow

### 7.1 Webhook Ingestion
**Component:** Azure Function (HTTP Trigger)

Steps:
1. Verify GitHub webhook signature
2. Validate event type and action
3. Extract repo, PR number, head SHA
4. Resolve org and policy profile
5. Generate idempotency key
6. Persist job state = RECEIVED
7. Enqueue ReviewJob to Service Bus

---

### 7.2 Orchestration
**Component:** Azure Durable Functions

Orchestrator controls the workflow:
1. Fetch PR data
2. Compute diff metrics
3. Run analyzers (parallel fan-out)
4. Normalize findings
5. Compute hotspots
6. Retrieve RAG context (optional)
7. Compute scorecard
8. Generate AI summary (optional)
9. Publish results
10. Mark job DONE or FAILED

---

### 7.3 PR Data Fetching
**Responsibilities**
- Fetch list of changed files
- Download diff/patch
- Capture PR metadata (labels, author)

**Storage**
- Raw artifacts stored in Blob Storage
- Metadata stored in Cosmos DB

---

### 7.4 Deterministic Analysis

#### Analyzer Types
- Security: Semgrep, custom rules
- Secrets: gitleaks
- Language linters (ESLint, SpotBugs, etc.)
- IaC scanners (future)

Each analyzer runs independently and outputs results in its native format.

---

### 7.5 Finding Normalization
All analyzer outputs are converted into a **canonical Finding format**.

Fields include:
- Category
- Severity
- Confidence
- Rule ID
- Location (file + line)
- Recommendation
- Policy references

This ensures tool-agnostic processing downstream.

---

### 7.6 Hotspot Detection

Hotspots are computed per file using:
- Lines added/deleted
- Complexity increase
- Security-sensitive paths
- Finding density and severity

Files are ranked and top-K hotspots are included in output.

---

### 7.7 RAG (Retrieval Augmented Generation)

When enabled:
- Internal standards and guidelines are indexed in Azure AI Search
- Relevant documents are retrieved using file paths + policy tags
- Retrieved context is referenced (not always sent to AI)

RAG never exposes private documents to external AI unless explicitly allowed.

---

### 7.8 Scoring Engine

Scoring is deterministic:
- Start from 100
- Deduct based on severity and category
- Apply category weights from PolicyProfile
- Evaluate quality gates (min score, max blockers)

Produces a Scorecard with pass/fail result.

---

### 7.9 AI Narration (Optional)

AI is used only for:
- Summarization
- Explanation
- Developer-friendly feedback

Modes:
- RULES_ONLY: AI disabled
- INTERNAL_ONLY: Internal LLM allowed
- EXTERNAL_REDACTED: Findings-only payload
- EXTERNAL_FULL: Explicit opt-in only

---

### 7.10 Publishing Results

**Preferred:** GitHub Check Run
- Status: success / failure
- Summary: scorecard + top findings
- Optional annotations

**Optional:** PR comment
- Markdown summary
- Hotspots and links to artifacts

---

## 8. Data Models

### 8.1 ReviewJob
- jobId
- repo
- prNumber
- headSha
- status
- policyProfileId
- timestamps

### 8.2 PolicyProfile
- enabled analyzers
- compliance flags
- scoring weights
- quality gates
- privacy mode
- allowed AI providers

### 8.3 Finding
- category
- severity
- confidence
- ruleId
- message
- location
- recommendation
- evidence reference

### 8.4 Scorecard
- overallScore
- categoryScores
- gateResults
- pass/fail

---

## 9. Storage Design

### Cosmos DB Containers
- jobs
- policies
- repo_config
- audit (optional)

### Blob Storage Layout
/artifacts/jobs/{jobId}/

Lifecycle policies enforce retention limits.

---

## 10. Extensibility

- New analyzers via plugin interface
- New SCMs via adapter layer
- New AI providers via LLM interface
- Policies editable via future Admin UI

---

## 11. Failure Handling

- Analyzer failure does not fail entire job
- Critical pipeline failures retry automatically
- DLQ used for unrecoverable errors
- Partial results may still be published (policy-driven)

---

## 12. Security Considerations

- Principle of least privilege
- No long-lived secrets
- Redaction before external calls
- Audit logging for compliance

---

## 13. Roadmap

### Phase 1
- GitHub App integration
- Deterministic analysis
- Scoring + Check Runs

### Phase 2
- RAG ingestion pipeline
- AI narration
- Admin UI

### Phase 3
- Azure OpenAI
- Advanced hotspot analytics
- Multi-SCM support

---

## 14. License

MIT
