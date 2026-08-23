# IAM Drills, Rebuilt on Okta
*Swapping Entra for Okta on the identity-specific pieces of JRT Life's drills. Defender and Purview stay as-is for threat posture and data risk, since those aren't IAM — say if you want those swapped too.*

Okta's current agentic identity platform is genuinely built around three questions worth memorizing verbatim, because <cite index="22-1">Okta's own framework for the secure agentic enterprise organizes the whole problem as: Where are my agents? What can they connect to? What can they do?</cite> That structure is a gift for an interview — it gives you a ready-made way to organize any answer about agent identity.

The platform itself: <cite index="22-1">Okta for AI Agents provides lifecycle management for agent identities — provisioning through a central registry, authentication using short-lived credentials rather than static API keys, and governance through policy controls that restrict what each agent can access and do</cite>. <cite index="23-1">It reached general availability on April 30, 2026</cite>.

---

## Drill 1 (Okta version) — The Three Questions, Applied to Each Agent

Answer all three for each of JRT Life's four agents. Don't let any answer be vague — "it can access what it needs" is not an answer.

| Agent | Where is it (registered)? | What can it connect to? | What can it do? |
|---|---|---|---|
| Intake Agent | | | |
| Underwriting Agent | | | |
| Claims Agent | | | |
| Coordinator Agent | | | |

**The cross-boundary case:** Claims Agent needs to call an external payment gateway — a system outside JRT Life's own tenant. <cite index="22-1">Okta introduced Cross App Access (XAA), an open protocol for standardizing how agents and applications connect securely across system boundaries</cite>, specifically because agents routinely need to reach resources across multiple platforms and organizational domains without a standard way to enforce authorization at that boundary.

Write one paragraph: why does Claims Agent calling an *external* payment gateway need a different authorization mechanism than Underwriting Agent calling JRT Life's *own* internal risk API? What does XAA solve that a same-tenant permission grant doesn't?

---

## Drill 2 (Okta version) — Lifecycle: Registry, Short-Lived Credentials, Kill Switch

**Birth:** Each agent is provisioned into Okta's central registry (built on Universal Directory). Write the registration entry for Claims Agent — what identity, what owner, what initial permission scope.

**Permissioning:** <cite index="20-1">Okta Identity Governance is built to enforce zero standing privileges</cite> — meaning no agent holds a permission it isn't actively using at that moment. Rewrite your Claims Agent disbursement policy from the earlier drills with this constraint made explicit: what does "zero standing privilege" mean for a payment call specifically — does Claims Agent request the payment-gateway permission fresh for every single claim, or does it hold a scoped grant for a session? Defend your answer.

**Credentials:** confirm this against your original JIT/sandbox drill — Okta's stated mechanism is <cite index="22-1">authentication using short-lived credentials rather than static API keys</cite>. Your generic drill assumed a PAM vault checkout per call; note where that matches Okta's real mechanism and where you were speculating beyond what's confirmed.

**Revocation:** <cite index="24-1">Okta's Universal Logout capability can function as a kill switch to terminate sessions associated with a misbehaving or compromised agent</cite>. Rewrite your Midnight Claim incident response (from the capstone) using this specific capability by name: what's the exact trigger condition that should fire Universal Logout on Claims Agent, and how fast does it need to happen given the exploit ran in about one minute?

**Decommissioning:** what should happen to Claims Agent's entry in the Universal Directory when it's retired — does it get deleted, or archived with its final permission state intact for audit purposes? Which is more defensible in a compliance review, and why?

---

## One Honest Gap to Practice Naming

<cite index="28-1">Okta signed a definitive agreement in July 2026 to acquire Permiso Security, a platform that detects threats across human, non-human, and agentic identities — with the deal expected to close around Okta's fiscal Q3 2027</cite>. That means Okta's own *threat-detection* capability for agent identities is still mid-acquisition, not fully integrated today.

**Practice this exact move for your interview:** if asked "does Okta handle threat detection for compromised agents the same way Defender does," the strong answer isn't a confident yes or no — it's naming the actual state: Okta's core strength today is identity lifecycle and access governance (registry, short-lived credentials, zero standing privilege, kill switch); deeper behavioral threat detection is being built via the Permiso acquisition and isn't fully landed yet. Saying that precisely, instead of guessing, is what separates someone who's read the marketing page from someone who's tracked the actual state of the product.

---

## Recall Check

- The three organizing questions: ______ / ______ / ______
- The protocol for cross-boundary agent-to-app access: ______
- The kill-switch capability for compromised agent sessions: ______
- The acquisition that will add deeper threat detection to Okta's agent platform: ______
