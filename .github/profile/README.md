# PreApply

## 1. Purpose

Modern infrastructure failures are rarely caused by syntax errors. They are caused by unintended blast radius, hidden coupling, and poor timing of otherwise‑valid changes. Existing tools tell engineers what will change, but not what could break.

The purpose of PreApply is to provide engineers with pre‑apply foresight by analyzing infrastructure changes deterministically and then using local AI to explain risk, impact, and safer alternatives in human terms.

This tool is not an automation engine. It is a decision‑support system designed to augment senior engineering judgment.

## 2. Problem Statement

Engineers today face several gaps when making infrastructure changes:

- Terraform plans show diffs, not consequences
- Blast radius is often discovered only after incidents
- Risk assessment relies on tribal knowledge
- Reviews focus on syntax, not system behavior
- Under pressure, engineers lack fast, reliable reasoning support

As a result, production outages frequently occur even when best practices are followed.

## 3. Design Principles

- **Deterministic First** – All risk facts must come from code and state, not AI guesses
- **Explainability** – Every risk must have a clear, inspectable reason
- **Local & Secure** – No infrastructure data leaves the engineer's environment
- **Engineer‑Centric UX** – CLI‑first, fast feedback, zero fluff
- **Judgment over Automation** – The tool advises; engineers decide

## 4. High‑Level Architecture

```
Code
Terraform Plan / State
        |
        v
Blast Radius Analyzer (Deterministic)
        |
        | Structured Risk Output (JSON)
        v
AI Reasoning Layer (Ollama via ollama-infra-cli)
        |
        v
Interactive CLI Chat
```

## 5. Core Components

### 5.1 Blast Radius Analyzer (Deterministic)

**Responsibilities:**

- Parse Terraform plan output (JSON)
- Build dependency graph of resources
- Identify shared infrastructure components
- Detect cross‑module and cross‑state coupling
- Compute blast radius score

**Outputs (example):**

```json
{
  "risk_level": "HIGH",
  "blast_radius_score": 82,
  "affected_components": ["shared-vpc", "payments-api", "auth-service"],
  "risk_factors": [
    "shared ALB modification",
    "cross-module dependency",
    "single NAT gateway"
  ],
  "recommended_actions": [
    "Isolate ALB per service",
    "Move VPC to core state",
    "Add AZ redundancy"
  ]
}
```

This layer contains no AI and no interpretation.

### 5.2 AI Reasoning Layer (Ollama Integration)

#### Access Model & Security Boundary

The AI layer does not have direct access to:

- Source code repositories
- Terraform files or state files
- CI/CD environments
- Cloud credentials

All infrastructure information is mediated by the deterministic analyzer. The AI receives only structured, pre‑computed analysis via standard input (stdin).

This design enforces:

- Determinism
- Security isolation
- Auditability
- Engineer trust

#### Responsibilities

- Consume structured risk output (JSON only)
- Translate risk into human‑readable explanations
- Answer engineer questions interactively
- Explicitly refuse to speculate beyond provided data

#### Data Flow

```
Analyzer Output (JSON)
        |
        | (stdin)
        v
ollama-infra-cli
        |
        v
Local LLM (no file system access)
```

#### Prompt Constraints

The AI is instructed to:

- Reason only from provided input
- Explain cause → impact → mitigation
- Avoid assumptions and hallucinations
- State clearly when information is insufficient

### 5.3 Interactive CLI Chat

**Example usage:**

```bash
preapply analyze plan.json --chat
```

**Example interaction:**

```
AI> This change has a high blast radius because it modifies a shared ALB
AI> used by three production services.

You> What is the worst-case impact?

AI> Requests may drop for auth-service during target re-registration,
AI> especially during peak traffic.

You> How can I reduce risk without a full refactor?

AI> Introduce a second ALB and migrate one service first.
```

## 6. Initial Scope (V1)

### Included

- Terraform plan parsing
- Dependency graph generation
- Shared resource detection
- Blast radius scoring
- AI explanation and chat

### Excluded

- Auto‑remediation
- CI/CD enforcement
- Runtime monitoring
- Cloud‑provider‑specific APIs beyond Terraform

## 7. Target Users

- Platform Engineers
- Senior DevOps Engineers
- Site Reliability Engineers (SREs)
- Cloud Architects

The tool assumes infrastructure literacy and does not target beginners.

## 8. Differentiation

| Existing Tools | PreApply |
|---------------|----------|
| Show diffs | Explain consequences |
| Detect syntax errors | Detect system risk |
| Reactive | Preventative |
| Generic | Context‑aware |

## 9. Future Extensions

- State coupling visualizer
- Failure mode enumeration
- Change‑safety scoring
- Historical incident correlation
- CI integration (advisory only)

## 10. Success Criteria

The project is successful if:

- Engineers can identify risky changes before apply
- Review quality improves measurably
- Postmortems reference PreApply outputs
- Engineers trust the tool's recommendations

## 11. Positioning Statement

PreApply is a local, deterministic risk analysis engine that helps engineers understand the real‑world impact of infrastructure changes before they reach production — with AI‑powered explanations, not AI guesses.

---

✨ This version swaps Infrastructure Risk Copilot (IRC) for PreApply everywhere, keeping the recruiter‑friendly polish intact.
