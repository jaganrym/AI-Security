# Designing & Implementing Policy-as-Code with Entra ID + OPA
### Governing Least-Privilege Tool Execution for Multi-Agent Systems

*A hands-on skill-building path, grounded in Microsoft Entra Agent ID (GA, May 2026) and Open Policy Agent (OPA)*

---

## Contents

1. [What's Now Real in Entra Agent ID](#1-whats-now-real-in-entra-agent-id)
2. [The Skill Stack](#2-the-skill-stack)
3. [Phase 1 — OPA & Rego Fundamentals](#3-phase-1--opa--rego-fundamentals-12-weeks)
4. [Phase 2 — OPA as PDP in an Architecture](#4-phase-2--opa-as-pdp-in-an-architecture-12-weeks)
5. [Phase 3 — Entra Agent ID as Your PIP](#5-phase-3--entra-agent-id-as-your-pip-23-weeks)
6. [Phase 4 — Multi-Agent Least-Privilege Tool Execution](#6-phase-4--multi-agent-least-privilege-tool-execution-23-weeks--portfolio-project)
7. [Phase 5 — CI/CD & Operational Maturity](#7-phase-5--cicd--operational-maturity-1-week)
8. [Suggested Timeline](#8-suggested-timeline)
9. [Interview-Ready Summary](#9-interview-ready-summary)

---

## 1. What's Now Real in Entra Agent ID

Microsoft Entra Agent ID went **generally available on May 1, 2026**, bringing purpose-built identity constructs for AI agents. Key pieces worth knowing cold:

| Concept | What it is |
|---|---|
| **Agent identity blueprint** | A template for creating individual agent identities with parent-child relationships, enabling consistent security policies across large numbers of agents — your mechanism for standardizing least-privilege scopes across an agent fleet |
| **Agent identity** | A special service principal in Microsoft Entra ID that a blueprint creates and is authorized to impersonate; it has no credentials of its own; its object ID / app ID are the stable identifiers used for authN/authZ decisions |
| **Agent's user account** | A second account (1:1 with the agent identity) for agents that need to access systems strictly requiring a user account, decorated as an AI agent |
| **Entra ID Auth SDK (sidecar)** | A sidecar pattern for agent authentication; used to integrate third-party agents (e.g., AWS Bedrock, n8n) via the sidecar or workload identity federation |
| **Protocol support** | OAuth 2.0, Model Context Protocol (MCP), and Agent-to-Agent (A2A) for authentication and communication |
| **Governance parity with humans** | Agents get the same identity-driven protections as users — adaptive access policies, real-time risk detection, lifecycle management, network-level controls — with all agent authentication and activity logged |

This matters because the **PIP layer isn't something you have to hack together** — Entra Agent ID is designed to *be* a rich attribute source (task scope, blueprint lineage, lifecycle state, risk signals) that OPA can query directly.

> Entra Agent ID is moving fast. Before building anything for production or a portfolio, pull the latest quickstart from `learn.microsoft.com/en-us/entra/agent-id/` rather than relying on a static summary.

---

## 2. The Skill Stack

```
Layer 4: Multi-Agent Governance    → task-scoped policies, delegation chains, blast-radius control
Layer 3: Identity Integration      → Entra Agent ID as the PIP (attribute source)
Layer 2: Policy-as-Code Engine     → OPA + Rego, testing, CI/CD
Layer 1: Foundations               → ABAC concepts, PDP/PEP/PIP/PAP
```

---

## 3. Phase 1 — OPA & Rego Fundamentals (1–2 weeks)

**Learn:**
- Install OPA locally; run it as a standalone server (`opa run --server`)
- Rego syntax: rules, sets, comprehensions, the `input` / `data` document model
- The **decision document** model — OPA returns whatever JSON structure you define, not just true/false (important for multi-agent systems where you need *reasons*, not just allow/deny)
- `opa eval`, `opa test`, unit testing policies with `_test.rego` files

**Hands-on exercise:**
Write a basic ABAC policy for a mock "file server" — allow read if `input.subject.clearance >= input.resource.sensitivity`. Write 5 unit tests: allow case, deny case, missing attribute, boundary case, deny-overrides case.

**Resource:** OPA's official "Policy Authoring" docs and the Rego Playground (play.openpolicyagent.org) — build muscle memory before adding infrastructure.

---

## 4. Phase 2 — OPA as PDP in an Architecture (1–2 weeks)

**Learn:**
- **OPA as a sidecar** vs. **centralized OPA service** — latency vs. consistency trade-offs
- **Bundles** — how OPA pulls policy from a remote source (Git, S3, OCI registry) so your PAP can push updates without redeploying OPA
- Integrating OPA behind an API gateway or service mesh (Envoy + OPA is the classic PEP/PDP pairing — study `opa-envoy-plugin`)
- **Decision logging** — every OPA decision can be emitted as your audit trail (directly counters "Repudiation & Untraceability" risk in agentic systems)

**Hands-on exercise:**
Stand up OPA in Docker, put a policy bundle in a local Git repo, configure OPA to poll it, and call OPA's REST API (`POST /v1/data/authz/allow`) from a script simulating a PEP.

---

## 5. Phase 3 — Entra Agent ID as Your PIP (2–3 weeks)

1. **Set up a dev tenant** and create an **agent identity blueprint** — your template for standardized least-privilege scopes across an agent fleet.
2. **Provision an agent identity** from that blueprint; inspect its object ID / app ID — these become the stable subject identifiers your Rego policies key off of.
3. **Instrument the Entra ID Auth SDK (sidecar)** in front of a mock agent — a pre-built PEP/token-acquisition layer you don't have to write from scratch.
4. **Pull agent attributes via Microsoft Graph** — blueprint lineage, owner/sponsor relationships, lifecycle state — and feed them into OPA as PIP data, either resolved into token claims upfront or queried by OPA at decision time via an HTTP data source.
5. Study how **Conditional Access / adaptive access policies** apply to agent identities the same way they do to users — this is your environmental/risk attribute source (sign-in risk, network context) feeding the same PDP.

**Hands-on exercise:**
Provision 2–3 agent identities from one blueprint, each simulating a different task role. Write Rego policies that key decisions off blueprint lineage plus a custom task-scope attribute, and verify agents from the same blueprint get a consistent baseline policy while task-specific claims narrow it further.

---

## 6. Phase 4 — Multi-Agent Least-Privilege Tool Execution (2–3 weeks) — Portfolio Project

### Architecture

```
Agent (Entra Agent ID identity, via sidecar)
   → requests tool call → PEP (wrapper around tool invocation)
                                ↓
                          OPA (PDP) ← policy bundle from Git (PAP)
                                ↓ (needs attributes)
                          PIP: Microsoft Graph (blueprint, lifecycle, risk)
                          PIP: Task Registry (current task scope)
                                ↓
                          Decision: allow/deny + reason → logged
                                ↓
                          PEP enforces
```

### Core Rego policy patterns

**Task-scoped least privilege + credential freshness + blueprint approval:**

```rego
package agent_authz

default allow = false

allow {
    input.subject.type == "agent"
    input.subject.blueprint_id == data.approved_blueprints[_]
    input.subject.lifecycle_state == "active"
    now := time.now_ns()
    now < input.subject.credential_expiry_ns

    task := data.task_registry[input.subject.task_id]
    input.action == task.allowed_tool
    input.resource.sensitivity <= task.max_sensitivity
}
```

**Tool-chaining detection** (counters the "email + database" exfiltration pattern):

```rego
package agent_authz

deny["blocked: high-risk tool chaining detected"] {
    input.action.risk_tier == "high"
    prior := data.session_history[input.subject.session_id]
    some t
    prior[t].risk_tier == "high"
    prior[t].tool != input.action.tool
}
```

**Lifecycle-state enforcement:**

```rego
package agent_authz

deny["blocked: agent lifecycle not active"] {
    input.subject.lifecycle_state != "active"
}
```

**Delegation depth limit** (counters delegation-loop attacks in multi-agent systems):

```rego
package agent_authz

deny["blocked: delegation chain too deep"] {
    input.subject.delegation_depth > data.limits.max_delegation_depth
}
```

### What to build

1. 2–3 mock agents provisioned as **real Entra agent identities**
2. A mock tool API (e.g., fake `send_email` and `query_database` functions)
3. OPA enforcing task-scoped policy on every tool call
4. A decision log (structured JSON) showing every allow/deny with its reason
5. **10–15 Rego unit tests** covering: expired credential, tool-chaining, blueprint mismatch, lifecycle-state deny, delegation-depth breach, and a deny-overrides conflict case

This project, done well, directly answers "how would you design PaC for multi-agent least privilege" with something concrete and demonstrable.

---

## 7. Phase 5 — CI/CD & Operational Maturity (1 week)

- GitHub Actions pipeline: `opa test` → `opa fmt --diff` (lint) → publish bundle to a registry on merge
- **Conflict-resolution testing** — write a test where two policies intentionally disagree and confirm your combining algorithm (e.g., deny-overrides) resolves it correctly
- `opa bench` for latency testing under load (matters — agent tool calls are latency-sensitive)
- Wire your OPA decision log alongside Entra's agent activity log for a unified audit trail

---

## 8. Suggested Timeline

| Weeks | Focus |
|---|---|
| 1–2 | Rego fundamentals, local testing |
| 3–4 | OPA as PDP in an architecture (Docker, bundles, decision logging) |
| 5–7 | Entra Agent ID integration as PIP (blueprints, sidecar, Graph attributes) |
| 8–10 | Build the multi-agent least-privilege reference project |
| 11 | CI/CD, conflict testing, benchmarking, portfolio polish |

---

## 9. Interview-Ready Summary

> "I used Entra Agent ID's blueprint model to standardize least-privilege scopes across an agent fleet, with OPA as the PDP evaluating task-scoped Rego policies against attributes pulled from Microsoft Graph — blueprint lineage, lifecycle state, and Conditional Access risk signals — enforced through the Entra ID Auth SDK sidecar acting as the PEP, with policy authored and tested in Git as the PAP."

### Next steps
- Pull the latest Entra Agent ID quickstart before implementing (`learn.microsoft.com/en-us/entra/agent-id/`)
- Provision a first agent identity blueprint in a dev tenant
- Write the full combined Rego module with all four patterns above, plus unit tests
