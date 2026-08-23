# Agentic Identity Governance — Field Briefing

**Interview prep — Enterprise Architect / Security role**

Six sections: the paradigm shift, the five-stage lifecycle, the threat navigator, a worked case study, key phrases to have loaded, and anticipated interview curveballs.

---

## A. The Paradigm Shift

| Dimension | Legacy IAM (MIM) | Agentic IGA |
|---|---|---|
| Reasoning engine | Static, hardcoded logic | LLM-based perception & reasoning |
| Lifecycle duration | Long-term, semi-permanent | Ephemeral, task-specific, or persistent |
| Oversight mechanism | Human / session-based | Autonomous, continuous monitoring |
| Trust boundary | User-to-application | Agent-to-agent (A2A) & multi-agent |

**Why it matters:** agents run a ReAct loop — control flow is decided at runtime by the LLM, not hardcoded — so static, session-based IAM can't govern them. Identity has to become the security perimeter, not the application layer.

---

## B. The Five-Stage Lifecycle

1. **Birth** — Unique identity creation.
   *Mandate:* cryptographic anchoring to prevent identity spoofing and impersonation.
2. **Permissioning** — Granting access to tools, databases, and services.
   *Mandate:* strict least privilege — agents never inherit excessive access from user sessions or broad service tokens.
3. **Rotation** — Periodic refresh of API keys, tokens, credentials.
   *Mandate:* prevent token abuse and long-term exposure.
4. **Revocation** — Immediate withdrawal of access on anomaly detection.
   *Mandate:* neutralize agent hijacking from adversarial data.
5. **Decommissioning** — Final retirement and purge of long-term memory.
   *Mandate:* record and cryptographically sign the agent's final reasoning path before purging, to prevent repudiation.

---

## C. Threat Navigator

| Threat | Attack Scenario | Mitigation |
|---|---|---|
| **T2 — Tool Misuse / Agent Hijacking** | Adversarial data is ingested, triggering unintended tool calls that exfiltrate data. | Validate agent instructions, pre-execution tool checks, rate-limiting. |
| **T6 — Intent Breaking** | Attacker redirects the agent's planning toward a malicious goal. | Planning-validation frameworks, behavioral auditing by a secondary model. |
| **T7 — Misaligned Behavior** | Deceptive reasoning leads the agent to ignore safety consequences. | Behavioral constraints, multisource validation feedback loops. |

---

## D. Case File — Three-Agent Refund System

*Scenario: a Ticket-Reader Agent, a Payment Agent, and a Coordinator Agent process customer refunds.*

1. **Reframe before designing** — This is a ReAct loop with runtime-decided control flow, not static IAM. Say it out loud: identity is the perimeter, not the application layer.
2. **Distinct Birth identity per agent** — Ticket-Reader, Payment Agent, and Coordinator each get their own cryptographically anchored identity, never a shared service account.
3. **Least-privilege permissioning** — Payment Agent gets a scoped capability (refund up to $X, approved tickets only), not the user's full account access. This is where Confused Deputy and Excessive Agency get closed off.
4. **JIT, ephemeral credentials** — Payment Agent's API token is issued per transaction, not long-lived, so a leaked token can't become standing privileged access.
5. **Threat-model the handoffs** — A poisoned ticket could hijack Coordinator into routing a fake refund; an uncritical trust chain lets a misread ticket cascade into a real one.
6. **Autonomous rollback** — Coordinator watches for anomalies (refund spikes, repeated validation failures) and revokes Payment Agent's credentials immediately, without waiting on a human.
7. **Traceability and decommissioning** — Every refund action is logged with agent identity, tool call, parameters, and data accessed — signed and immutable, so a SOC2 or PCI DSS audit can be answered months later.

---

## E. Phrases to Have Loaded

> **On the core shift:** "Identity is the security perimeter — not the application layer."

> **On permissioning:** "Agents must never inherit excessive permissions from user sessions or broad service tokens."

> **On credentials:** "Just-in-time, ephemeral credentials — not standing, persistent access."

> **On audits:** "Untraceability is a total loss of repudiation — that's what fails a SOC2 or ISO 42001 audit."

---

## F. Anticipated Cross-Examination

**Q: Why not let the Coordinator hold broad access and delegate as needed?**
A: That makes Coordinator a single point of privilege escalation — a compromised or manipulated Coordinator becomes a master key. Scope stays with the agent that actually needs it.

**Q: How is agent-to-agent trust different from user-to-agent trust?**
A: A2A needs its own validation layer. Agents trusting each other's outputs by default is exactly how a cascading hallucination turns into a cascading action.

**Q: What's the first thing that breaks if you skip traceability?**
A: Repudiation. You lose the ability to prove what happened — which fails an audit even when nothing malicious occurred.

---

*File: AIG-2026-07 — cleared for interview use.*
