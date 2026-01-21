# PolicyPR

**PolicyPR** is a **policy-driven, privacy-first pull request review platform** that integrates directly with GitHub.

It automatically reviews pull requests using **deterministic analysis**, **organizational policies**, and **optional AI assistance** to deliver **consistent, explainable, and enterprise-ready code reviews** — without requiring source code to leave your environment.

---

## 🚀 What PolicyPR Does

PolicyPR reviews pull requests and provides:

- 🔍 **Structured findings** (security, reliability, maintainability, compliance)
- 🔥 **Hotspot detection** for high-risk files and changes
- 📊 **Quality scores & pass/fail gates**
- 📜 **Policy-aware feedback** aligned to org standards (HIPAA, GDPR, OWASP, NFRs)
- 🧠 **Optional AI-generated explanations** (privacy-controlled)

All results appear **directly inside GitHub pull requests** as:
- Pull request comments
- GitHub Check Runs (status, score, and blocking issues)

---

## 🧩 What PolicyPR Is (and Is Not)

### ✅ What it is
- A **GitHub App** integrated tool
- **Hosted externally** (Azure serverless)
- Event-driven and scalable
- Designed for **enterprise and regulated environments**

### ❌ What it is not
- Not a browser extension
- Not a GitHub UI plugin
- Not a black-box AI reviewer

---

## 🏗️ High-Level Architecture

PolicyPR runs as a **serverless, event-driven platform**.

### Flow
1. Pull request event occurs in GitHub
2. GitHub App webhook triggers PolicyPR
3. PR metadata and diff are fetched securely
4. Analysis pipeline runs:
   - Static analyzers
   - Policy checks
   - Hotspot detection
   - Deterministic scoring
5. Optional AI explanation layer runs
6. Results are posted back to GitHub

---

## 🔐 Privacy-First by Design

PolicyPR is built with **data protection as a core principle**.

- ❌ No source code is sent to external AI models by default
- ✅ Deterministic analysis runs fully inside Azure
- ✅ AI usage is **policy-controlled**
- ✅ Supports multiple privacy modes:
  - Rules-only (no AI)
  - Internal-only AI (Azure OpenAI / self-hosted)
  - External AI (metadata-only, redacted)
- ✅ Full auditability of analysis steps

---

## 📐 Policy-Driven Reviews

Each organization or repository defines a **Policy Profile** that governs reviews.

Policies can include:
- Security posture (OWASP, secrets, encryption)
- Compliance requirements (HIPAA, GDPR, PCI)
- Architecture preferences (Clean Architecture, Hexagonal)
- Non-functional requirements (performance, reliability, observability)
- Scoring weights and quality thresholds
- Allowed AI providers and data-sharing modes

This ensures **consistent enforcement across teams and repos**.

---

## 🔎 Findings & Hotspots (No AI Required)

PolicyPR produces findings using **deterministic techniques**:

### Findings sources
- Static analyzers (e.g., Semgrep, linters, security scanners)
- Custom organization rules
- PR diff heuristics
- Test coverage heuristics
- Secrets detection

### Hotspot detection
Files are ranked using:
- Change size and complexity
- Security-sensitive paths
- Historical churn signals
- Severity and density of findings

This makes reviews **repeatable, explainable, and auditable**.

---

## 📊 Scoring & Quality Gates

Each pull request receives:
- Category scores (Security, Reliability, Maintainability, etc.)
- Overall quality score
- Pass/fail decision based on policy thresholds

Scoring is **deterministic**, not subjective AI output.

---

## 🧠 AI — Optional and Controlled

AI in PolicyPR is an **assistive layer**, not the source of truth.

When enabled, AI is used to:
- Summarize findings
- Explain risks in developer-friendly language
- Suggest remediation patterns

AI never replaces:
- Deterministic findings
- Policy rules
- Scoring logic

---

## 🌐 Technology Stack

- **Azure Functions** (serverless compute)
- **Azure Service Bus** (event queue)
- **Durable Functions** (workflow orchestration)
- **Cosmos DB** (job state, policies)
- **Azure Blob Storage** (artifacts)
- **Azure AI Search** (RAG / vector search)
- **GitHub App APIs**
- **Python & TypeScript**
- **Pluggable LLM providers** (Gemini, Azure OpenAI, others)

---

## 🎯 Who Is PolicyPR For?

- Engineering teams in regulated industries
- Organizations needing consistent PR quality enforcement
- Teams adopting AI cautiously and responsibly
- Platform teams building internal developer tooling

---

## 📌 Project Status

🚧 **Active development / reference architecture**

Initial focus:
- GitHub App integration
- Azure serverless pipeline
- Deterministic findings & scoring
- Policy framework
- Optional AI narration layer

---

## 📜 License

MIT
