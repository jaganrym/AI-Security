# Practicing the Skill Drills on the Microsoft Stack
*JRT Life's four agents (Intake, Underwriting, Claims, Coordinator), mapped onto Agent 365, Entra, Defender, and Purview*

Your table maps cleanly onto the four drills you already did in pseudocode. The mapping isn't a coincidence — Microsoft built these four products as one coordinated system, with Agent 365 as the control plane sitting on top of the other three. This companion asks you to redo each drill *in terms of what these products actually do*, so you can talk about the architecture in real product language, not just generic security terms.

| Your Drill | Generic Concept | Real Microsoft Product |
|---|---|---|
| Drill 1 — Policy-as-Code / TLAC | Function-level authz at each reasoning step | **Entra** (agent RBAC, conditional access) + **Agent 365** (policy enforcement) |
| Drill 2 — JIT / Birth attestation | Agent identity lifecycle, ephemeral credentials | **Entra Agent ID** (agent directory + lifecycle) + **Agent 365** (registry + certification) |
| Drill 3 — Confused Deputy / anomaly | Threat detection, behavioral baselines | **Defender** (threat posture for AI) + **Purview** (insider risk triage) |
| Drill 4 — A2A trust / consensus | Cross-agent authentication, data-boundary enforcement | **Agent 365** (agent-to-agent via MCP) + **Purview** (data access boundaries) |

---

## Drill 1 (Redone) — Identity: Entra as the Policy Layer

**What's real:** <cite index="16-1">Entra Agent ID provides a unified directory of all agent identities, letting you track their lifecycle, manage permissions, and secure their access to organizational resources</cite>.

**Task:** For each of JRT Life's four agents, write what its Entra Agent ID entry would need to contain to make your Drill 1 policy enforceable:

| Agent | Registered scope (what Entra should allow) | What Entra should explicitly deny |
|---|---|---|
| Intake Agent | | |
| Underwriting Agent | | |
| Claims Agent | | |
| Coordinator Agent | | |

**The real design question:** your original Drill 1 policy checked `input.claim.source != "unverified_chat_annotation"` to block the Midnight Claim exploit. Where does that check actually live in this stack — is it an Entra conditional-access rule, an Agent 365 policy, or does it have to live in your own application code because it's too claim-specific for a platform-level control? Argue your answer; this is exactly the kind of question a Microsoft-stack interviewer asks to see if you understand what the platform does versus what you still have to build yourself.

---

## Drill 2 (Redone) — Lifecycle: Agent 365 Registry as Birth/Decommission

**What's real:** <cite index="6-1">the Agent 365 registry, powered by Entra Agent ID, is a single source of truth for every agent in the organization</cite>, and Agent 365 integrates with Purview to apply data protection and with Defender to detect threats in real time.

**Task:** Walk through what "Birth" and "Decommissioning" look like concretely for Claims Agent in this stack:

- **Birth:** Claims Agent gets registered in the Agent 365 registry with an Entra Agent ID. What ownership metadata should be attached at this step (who owns it, what data it's certified to touch)? JRT Life has no dedicated security architect yet — who *should* be the registered owner, practically?
- **Decommissioning:** if Claims Agent is ever retired or replaced, what needs to happen in the registry so it doesn't become a "shadow agent" — the exact risk the registry exists to prevent?

**Compare to your pseudocode drill:** your JIT/sandbox sequence assumed ephemeral tokens injected per-call. Note honestly: the search results confirm Entra Agent ID handles *identity and lifecycle tracking*, but don't confirm a specific per-call ephemeral-token mechanism for Agent 365 the way CyberArk-style PAM does. When you don't have a confirmed platform feature, say so in an interview rather than assuming the product does everything your generic model called for — that intellectual honesty matters more than sounding complete.

---

## Drill 3 (Redone) — Oversight: Defender + Purview as the Detection Layer

**What's real:** Microsoft Defender's <cite index="13-1">threat protection for AI services provides insights into threats that might affect generative AI applications, including the option to surface suspicious segments directly from user prompts for triage</cite>. On the data side, <cite index="14-1">Purview offers a Data Security Triage Agent for DLP and Insider Risk Management that analyzes content and intent and prioritizes the highest-risk activity</cite>.

**Task — rebuild your Confused Deputy check as a real detection story:**

1. In the Midnight Claim scenario, the malicious instruction was buried inside the applicant's chat text. Which product would actually have a chance of catching that at the prompt level — Defender's prompt-evidence capture, or a Purview DLP rule? Explain the difference in *what each one is looking at* (Defender: the prompt/response content itself; Purview: sensitivity of data and insider-risk behavioral signals).
2. Rewrite your Drill 3 anomaly table using Purview's actual triage categories — <cite index="14-1">User Risk (behavioral anomalies), File Risk (sensitivity and activity history), and Activity Risk (a combination of file, device, and app indicators)</cite> — instead of the generic metrics you invented before.

| Purview Risk Category | JRT Life example for Claims Agent |
|---|---|
| User Risk | |
| File Risk | |
| Activity Risk | |

---

## Drill 4 (Redone) — Trust Boundary: Agent 365 as the A2A Layer

**What's confirmed:** Agent 365 exposes agent-to-agent interactions through consistent MCP interfaces, and ties agent activity back to Purview for data protection and Defender for threat response — but the specific mutual-TLS or multi-agent consensus mechanics from your original Drill 4 aren't something the current public material spells out in detail.

**Task:** Since this is the one area where the platform's exact mechanics aren't fully public yet, treat it as a design proposal instead of a lookup:

- Write, in one paragraph, how you'd *pitch* a consensus requirement for JRT Life's >$10,000 disbursements to a Microsoft account team — what would you ask Agent 365 and Purview to jointly enforce, given what they're confirmed to already do (identity registry + data boundaries)?
- Flag explicitly: is this a gap in the current product, or a gap in your own research? That distinction is exactly what a good architect states out loud in a real design review instead of guessing silently.

---

## Recall Check

Fill in the blank without looking back:

- "\_\_\_\_\_ is the control plane for AI agents" → ______
- The registry that gives you agent ownership, deployment platform, and permissions is powered by ______
- The product that surfaces prompt-level evidence for AI threat triage is ______
- The product that triages DLP and Insider Risk alerts using User/File/Activity risk categories is ______
