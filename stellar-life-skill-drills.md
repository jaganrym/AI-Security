# Practical Drills: Agentic IGA Skill-Building Blueprint
*Applied to JRT Life's four-agent system (Intake, Underwriting, Claims, Coordinator)*

Each drill asks you to actually produce the artifact the blueprint names — not describe it. That's the difference between reciting "I'd use JIT credentials" in an interview and being able to sketch the sequence when someone says "show me."

---

## Drill 1 — Contextual Policy-as-Code (Dimension 1)

**Task:** Claims Agent is about to call the `disburse_payment` tool. Write the policy check that must pass *at that specific reasoning step* — not a static role, an intent + tool-boundary check.

```rego
# Pseudocode in the style of Open Policy Agent (Rego)
package JRTlife.claims

default allow_disbursement = false

allow_disbursement {
    input.agent_identity == "claims-agent"
    input.tool == "disburse_payment"
    input.claim.status == "pre_approved_by_rules_engine"
    input.claim.amount <= input.agent_scope.max_disbursement
    input.claim.source != "unverified_chat_annotation"   # blocks the Midnight Claim exploit
    input.credential.type == "ephemeral"
    input.credential.ttl_remaining_seconds > 0
}
```

**Now do it yourself:** write the equivalent policy for Underwriting Agent calling the risk-scoring API. What field would you add that has no equivalent in the Claims policy? *(Hint: pricing decisions need a different kind of boundary than payments — think about what "intent" means for underwriting versus disbursing.)*

**Tool-Level Access Control (TLAC) check:** before writing the policy, answer — what's the cryptographic assertion Claims Agent must present to prove it's authorized *at this step*, not just in general? (e.g., a short-lived signed token scoped to `disburse_payment` + this specific claim ID, not a standing API key.)

---

## Drill 2 — JIT Credential & Sandbox Lifecycle (Dimension 2)

**Task:** Sequence the exact steps from "Claims Agent decides to pay" to "container destroyed," for one disbursement.

```
1. Claims Agent's reasoning loop concludes: disburse $X to claim #Y
2. Coordinator receives the tool-call request, runs Drill 1's policy check
3. IF allowed → identity broker (e.g. CyberArk-style PAM) checks out a
   short-lived token scoped ONLY to: {tool: disburse_payment, claim_id: Y, ttl: 30s}
4. A transient sandbox container spins up with restricted CPU/memory
5. The ephemeral token is injected into the container — never into the
   agent's persistent memory or logs
6. The payment API call executes inside the sandbox
7. Result (success/fail) returns to Coordinator
8. Container is destroyed immediately; token expires or is revoked regardless
9. Traceability log entry written (see the capstone's Milestone 7 schema)
```

**Now do it yourself:** where in this sequence would you insert the "Birth attestation" check — does an agent need to re-attest its identity every single call, or only periodically? Write your reasoning, not just an answer — this is a real design trade-off (latency vs. security) an interviewer will push on.

---

## Drill 3 — Confused Deputy Check + Anomaly Baseline (Dimension 3)

**Task A — Confused Deputy mitigation:** Underwriting Agent has elevated access to the risk-pricing engine. Write the check that stops it being manipulated into pricing a policy on behalf of an unauthorized third party.

```
BEFORE calling risk_scoring_api:
  verify: does the end-user (applicant) in THIS conversation match the
          user_id attached to the original authenticated session?
  IF mismatch → halt, do not call tool, escalate to Coordinator
```

Write one sentence explaining *why* this check has to happen at every tool invocation, not just once at session start. *(This is the exact question from the JRT Life capstone's Milestone 5 — if your answer here doesn't match your policy-as-code artifact from that exercise, reconcile them.)*

**Task B — Behavioral anomaly baseline:** Define a "normal" pattern and a flaggable deviation for Claims Agent.

| Metric | Normal baseline | Anomaly trigger |
|---|---|---|
| Claims processed per hour | *(you fill in a plausible number)* | *(you fill in a threshold)* |
| Disbursement approved outside business hours | Rare, low-dollar only | *(what's your threshold?)* |
| Recursive tool calls per claim | 1 (single payment call) | *(what would recursive calls even mean here — is 2+ always bad?)* |

---

## Drill 4 — A2A Mutual Auth + Multi-Agent Consensus (Dimension 4)

**Task A:** Sketch the handshake before Coordinator forwards a claim to Claims Agent.

```
Coordinator → Claims Agent: signed request {claim_id, timestamp, coordinator_signature}
Claims Agent verifies: signature valid? timestamp fresh (not replayed)?
                        sender identity == known Coordinator identity?
Claims Agent → Coordinator: signed acknowledgment
```

**Task B — Consensus for high-risk operations:** The blueprint says high-risk actions need multiple independent agents to sign off. Design this for a disbursement over $10,000 at JRT Life.

Write out: which second agent (or human) has to co-sign, what they're actually verifying (not just rubber-stamping the first agent's decision), and what happens if they disagree. *(A design that lets the second signer see only "approve/deny" without the reasoning isn't real consensus — decide what they actually get to review.)*

---

## Recall Check — The Three Interview Framing Lines

Before moving on, try reconstructing these from memory, then check yourself:

1. The MIM-vs-runtime-identity line (starts with "MIM was architected for...")
2. The sandboxing line (starts with "I don't just secure the API...")
3. The governance line (starts with "In an agentic ecosystem...")

If you can't reconstruct the core idea of each (not word-for-word — that's not the point), reread Dimension 1–4's "skills to build" once more before your next drill session.
