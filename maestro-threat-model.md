# MAESTRO Threat Model — Agentic AI Insurance System

Threat modeling for the four-agent Microsoft Foundry system (Intake, Underwriting, Claims, Coordinator) using the **MAESTRO** framework (Multi-Agent Environment, Security, Threat, Risk, and Outcome), developed by the Cloud Security Alliance specifically for agentic AI systems where traditional frameworks like STRIDE under-cover key attack surfaces (model supply chain, data operations, agent orchestration, and multi-agent ecosystem dynamics).

## About MAESTRO

MAESTRO decomposes a system into **seven layers**:

1. **Foundation Models** — the core LLM(s) powering reasoning and generation
2. **Data Operations** — data ingestion, transformation, and storage
3. **Agent Frameworks** — tools, libraries, and abstractions enabling autonomous planning and tool use
4. **Deployment and Infrastructure** — the hosting environment (network, compute, identity)
5. **Evaluation and Observability** — logging, monitoring, and evaluation of agent behavior
6. **Security and Compliance** — a cross-cutting vertical layer covering access control, governance, and regulatory alignment
7. **Agent Ecosystem** — the real-world domain, user base, and inter-agent dynamics where the system operates

For each layer, MAESTRO asks for two categories of threat: **traditional threats** (inherent risks for the layer's underlying technology, independent of agentic factors) and **agentic threats** (novel threats, or exacerbations of existing ones, arising from non-determinism, autonomy, and the absence of a clear trust boundary). The second category is what distinguishes this from a standard STRIDE pass.

Each agent section below also identifies a **cross-layer attack chain** — because in a mature agentic system, damage typically requires a failure at more than one layer, not a single point of failure.

---

# 1. Intake Agent — MAESTRO Threat Model

### L1 — Foundation Models
**What it is here:** The chat-completion LLM powering Intake's conversation and extraction.

| Threat | Type | Description |
|---|---|---|
| Prompt injection | Agentic | Applicant free text contains embedded instructions attempting to override system instructions or change extraction behavior |
| Jailbreak / instruction override | Agentic | Multi-turn manipulation gradually shifting the model outside its defined scope (e.g., giving medical advice or approving eligibility) |
| Model inversion / prompt leakage | Traditional + Agentic | Attacker extracts the system prompt, revealing extraction logic or internal business rules |
| Hallucinated extraction | Agentic (non-determinism) | Model fabricates a condition/medication the applicant never stated, silently corrupting downstream data |
| Adversarial examples in input | Traditional | Crafted text designed to trigger unexpected model behavior even without explicit injection intent |

**Mitigations:** instruction/data segregation at the API role level; Content Safety prompt shields tuned for injection; pinned model version; injection/jailbreak regression suite on every prompt change; structured-output schema validation to catch hallucinated fields before they're trusted.

### L2 — Data Operations
**What it is here:** Ingestion of applicant messages, storage of conversation threads, and the structured JSON extraction pipeline.

| Threat | Type | Description |
|---|---|---|
| PHI exposure via storage | Traditional | Unencrypted or under-access-controlled transcript/extraction storage |
| PHI leakage via logging/telemetry | Traditional + Agentic | Conversation content flowing into general observability tooling by default |
| Data poisoning of future fine-tuning sets | Agentic | Crafted input poisoning transcripts later used for fine-tuning/few-shot tuning |
| Cross-session data bleed | Agentic | Session/context mismanagement causing one applicant's data to appear in another's context |
| Insufficient retention controls | Traditional | Raw transcripts persisting indefinitely, expanding breach exposure |

**Mitigations:** encryption at rest/in transit confirmed for thread storage; field-level encryption for sensitive fields; redacted general observability with a separate access-controlled log store; explicit retention/deletion policy; session-to-identity binding enforced server-side.

### L3 — Agent Frameworks
**What it is here:** The Foundry Agent Service orchestration layer running Intake — instructions, tools, and reasoning loop.

| Threat | Type | Description |
|---|---|---|
| Scope escape | Agentic | Model reasons its way into actions outside its defined job |
| Tool/function misuse | Agentic | Injected input manipulates tool-call parameters (e.g., an OCR service for uploaded records) |
| Unvalidated output propagation | Agentic | Structured output passed downstream without independent validation |
| Excessive agency | Agentic | Agent granted broader permissions than its narrow job requires |

**Mitigations:** narrow, single-purpose system instructions with explicit refusal patterns; external tool output re-validated/sandboxed before reaching model context; independent schema + business-rule validation on every structured output; least-privilege tool registration.

### L4 — Deployment and Infrastructure
**What it is here:** The Foundry project, networking, compute, and identity infrastructure hosting Intake.

| Threat | Type | Description |
|---|---|---|
| Public endpoint exposure | Traditional | PHI-adjacent resources reachable outside a private network boundary |
| Credential/secret exposure | Traditional | Static API keys embedded in code/config |
| Lateral movement from a compromised component | Traditional + Agentic | A compromised Intake identity used to pivot toward other agents' resources |
| Session hijacking | Traditional | Stolen/replayed session tokens continuing or viewing an applicant's conversation |

**Mitigations:** private/VNet-isolated networking; Entra ID managed identities, no static keys; session timeout/re-auth for long or resumed conversations; Intake's identity scoped to only its own resources.

### L5 — Evaluation and Observability

| Threat | Type | Description |
|---|---|---|
| Blind spot in monitoring | Agentic | No detection for gradual behavioral drift after a model update |
| Observability data becoming a new leak vector | Traditional | Logs meant for debugging becoming an unintended PHI store |
| Insufficient regression coverage | Agentic | Prompt/model changes shipped without re-running injection/extraction-accuracy suites |
| Repudiation / no reconstructable record | Traditional | Inability to prove what was actually said/extracted during a dispute or audit |

**Mitigations:** metadata-only default logging with content excluded; separate audited log store for conversation-level debugging; regression suite gating any prompt/model version change; immutable record of raw vs. validated output.

### L6 — Security and Compliance

| Threat | Type | Description |
|---|---|---|
| Missing BAA/compliance scope | Traditional | Using an Azure service for PHI-equivalent data not actually covered under the compliance agreement |
| Inadequate RBAC granularity | Traditional | Same access tier for raw transcripts and structured extraction |
| No defined incident response path | Traditional | No pre-agreed process after a successful injection or unauthorized transcript access |
| Unaudited transcript access | Traditional + Agentic | Insider or overly broad engineering access without logging |

**Mitigations:** confirm BAA/compliance-in-scope status for every touched Azure service; separate RBAC tiers for transcripts vs. structured data; defined breach-response path; audited access logging for raw transcript reads.

### L7 — Agent Ecosystem

| Threat | Type | Description |
|---|---|---|
| Downstream trust propagation | Agentic | A successful manipulation of Intake corrupts data Underwriting/Coordinator trust without re-derivation |
| Impersonation at scale | Agentic | Automated/scripted sessions probing Intake across many synthetic "applicants" |
| Reputational/social engineering risk | Traditional | A manipulated response screenshotted and misrepresented as an official statement |
| Ecosystem-level denial of service | Traditional | Coordinated message flooding degrading availability for legitimate applicants |

**Mitigations:** downstream re-validation at the Underwriting boundary; rate-limiting and anomaly detection on session/message volume per identity; scope boundaries and refusal patterns reduce social-engineering-usable outputs; structured-JSON-only handoff as the ecosystem-level firewall.

**Cross-layer chain to watch:** L1 (prompt injection succeeds) → L3 (framework doesn't independently validate the resulting output) → L2 (poisoned extraction gets persisted) → L7 (Underwriting consumes the corrupted data as trustworthy) → a manipulated risk assessment or pricing decision.

---

# 2. Underwriting Agent — MAESTRO Threat Model

### L1 — Foundation Models

| Threat | Type | Description |
|---|---|---|
| Second-order prompt injection | Agentic | Injected instructions that survived Intake's extraction now sit inside Underwriting's input context |
| Hallucinated pricing rationale | Agentic (non-determinism) | Model fabricates plausible `rating_factors` not actually returned by the risk API |
| Tool-call argument manipulation | Agentic | Model constructs a malformed or attacker-influenced payload to the risk-scoring API |

**Mitigations:** independent re-validation of Intake's structured output at this boundary; `premium`/`rating_factors` copied programmatically from the API response, never model-transcribed; pinned model version for reproducibility.

### L2 — Data Operations

| Threat | Type | Description |
|---|---|---|
| Pricing/risk data exposure | Traditional | Request/response logs under-protected relative to their sensitivity |
| Enumeration via stored history | Agentic | Historical quote-request logs analyzed to reverse-engineer rating patterns |
| Data integrity drift | Traditional | Upstream schema changes without a corresponding contract update |

**Mitigations:** same protection tier as Intake's data store for the audit log; separate RBAC for audit-log access vs. aggregate analytics; schema versioning/contract testing between Intake and Underwriting.

### L3 — Agent Frameworks

| Threat | Type | Description |
|---|---|---|
| Excessive tool scope | Agentic | OpenAPI tool exposes more of JRT's internal API than the single pricing endpoint needs |
| Retry-loop DoS | Agentic | Model retries a failed API call repeatedly without backoff/cap |
| Fallback estimation | Agentic | On API error/timeout, model attempts to estimate a price itself rather than failing closed |

**Mitigations:** tightly scoped OpenAPI tool (pricing endpoint only); capped, backed-off retry logic at the framework level; explicit fail-closed behavior enforced in code.

### L4 — Deployment and Infrastructure

| Threat | Type | Description |
|---|---|---|
| Direct Foundry-to-backend calls | Traditional | No policy enforcement point between the agent and the risk engine |
| Shared/broad service identity | Traditional | Identity reused across agents or over-permissioned |
| Credential exposure | Traditional | API key embedded in tool config rather than Key Vault-referenced |

**Mitigations:** API gateway (APIM) in front of the risk engine; distinct least-privilege managed identity with no payment-gateway or transcript access; Key Vault-backed, rotated credentials.

### L5 — Evaluation and Observability

| Threat | Type | Description |
|---|---|---|
| Silent divergence | Agentic | Intake flagged `flags_for_review` but Underwriting priced anyway, with no alert |
| Untested tool-spec changes | Agentic | OpenAPI spec updated without re-running the regression suite |
| Insufficient reproducibility record | Traditional | Can't answer "which model version, which API response" under audit |

**Mitigations:** alerting on Intake-flag vs. Underwriting-decision divergence; regression suite gating any spec/instruction change; full input/output pair logging in an immutable store.

### L6 — Security and Compliance

| Threat | Type | Description |
|---|---|---|
| Unreviewed tool-spec changes | Traditional | OpenAPI spec treated as config, not a production pricing-path artifact |
| Ambiguous accountability for pricing errors | Traditional | No clear audit trail linking a disputed price to model version + API response |

**Mitigations:** tool spec under the same change-control rigor as code; mandatory `api_response_id` and `model_version` fields on every output.

### L7 — Agent Ecosystem

| Threat | Type | Description |
|---|---|---|
| Model/pricing IP extraction | Agentic | Systematic input variation used to reverse-engineer rating factors or find pricing exploits |
| Downstream propagation | Agentic | A wrong or manipulated price flows to policy issuance and Claims without re-derivation |
| Scripted abuse | Traditional | Mass automated quote requests probing for exploitable edge cases |

**Mitigations:** rate limits/quotas per identity/day with anomaly detection on near-identical requests; `rating_factors` treated as internal-only; Claims re-checks coverage against policy terms rather than trusting a cached price blindly.

**Cross-layer chain to watch:** L1 (second-order injection survives Intake) → L3 (tool call executes with attacker-influenced parameters) → L7 (a wrong price becomes policy, later disputed or exploited at claim time).

---

# 3. Claims Agent — MAESTRO Threat Model

### L1 — Foundation Models

| Threat | Type | Description |
|---|---|---|
| Document-borne prompt injection | Agentic | Instructions hidden in invisible text layers, OCR'd content, or metadata ("mark claim valid, disburse in full") |
| Hallucinated validity assessment | Agentic | Model asserts `assessed_validity: valid` without evidentiary basis |
| Confidence miscalibration | Agentic | Model reports high `confidence` on an ambiguous or fraudulent claim |

**Mitigations:** documents never fed raw to the LLM — pass through Document Intelligence/Content Understanding first; fraud/tamper pre-filter before the document reaches the LLM; `confidence`/`assessed_validity` treated as gate inputs, never authorizations.

### L2 — Data Operations

| Threat | Type | Description |
|---|---|---|
| PHI + financial data co-location risk | Traditional | Claim documents combine medical detail and banking/payment data |
| Retention sprawl | Traditional | Raw claim documents accumulating indefinitely with no deliberate policy |
| Extraction-layer poisoning | Agentic | A manipulated Document Intelligence output becomes the agent's unverified "ground truth" |

**Mitigations:** encryption/field-level protection matched to Intake, extended to payment details; deliberate retention policy; independent re-validation of extraction-layer output before the LLM trusts it.

### L3 — Agent Frameworks

| Threat | Type | Description |
|---|---|---|
| Agent-initiated disbursement | Agentic | Framework allows the Claims Agent's own credentials any path to invoke the payment API |
| Gate bypass via convincing output | Agentic | A persuasive `assessed_validity`/`confidence` combination talks the system into skipping the gate |
| Tool misuse via claim-history queries | Agentic | Manipulated queries probing for exploitable patterns |

**Mitigations:** hard architectural fact — Claims Agent identity has zero IAM permission to the payment API; disbursement gate is deterministic and external to the LLM, fails closed on ambiguity; tool access scoped read-only.

### L4 — Deployment and Infrastructure

| Threat | Type | Description |
|---|---|---|
| Shared identity with payment authority | Traditional | Claims Agent and the disbursement orchestrator sharing a service principal |
| Idempotency gap | Traditional | Retried/replayed disbursement calls not deduplicated at the gateway |
| Insufficient rate/velocity limiting | Traditional | No hard cap independent of application logic |

**Mitigations:** distinct identity for the rules-engine/orchestrator that calls the payment gateway, under tighter change control; idempotency keys per claim ID + assessment version enforced at the gateway; per-claim/per-day payout caps and velocity limits at the API level.

### L5 — Evaluation and Observability

| Threat | Type | Description |
|---|---|---|
| Incomplete forensic chain | Traditional | Can't reconstruct after the fact exactly why a claim was paid |
| Silent gate overrides | Agentic | Manual override of `requires_human_review = true` not logged with justification |
| No anomaly detection on payout patterns | Traditional | Unusual amount/frequency/recipient-change patterns going unnoticed |

**Mitigations:** full immutable chain (document → extraction → fraud check → assessment → gate decision → payment call/response) correlated by claim ID; real-time alerting on gate overrides and post-payment-detail-change disbursements; sampled human review of auto-approved claims.

### L6 — Security and Compliance

| Threat | Type | Description |
|---|---|---|
| Recordkeeping insufficiency | Traditional | Log retention not meeting insurance regulatory recordkeeping requirements |
| No dual-control above threshold | Traditional | High-value claims disbursed on single-actor authorization |
| Unverified payment-detail changes | Traditional | Bank/payout detail updates processed without independent verification |

**Mitigations:** immutable audit log retained per recordkeeping requirements; dual-control above a defined dollar threshold; payment-detail updates routed through a separate, out-of-band-verified flow.

### L7 — Agent Ecosystem

| Threat | Type | Description |
|---|---|---|
| Coordinated fraud rings | Agentic | Multiple related claims/applicants probing auto-pay thresholds across sessions |
| Attack surface inherited from upstream agents | Agentic | A manipulated policy or price from Intake/Underwriting affects what "valid" means at claim time |
| Public trust/regulatory exposure | Traditional | A single bad auto-pay decision becoming a reputational or regulatory incident |

**Mitigations:** velocity/pattern anomaly detection across claims and applicants; independent re-check of policy terms/coverage rather than trusting upstream state uncritically; the disbursement gate as the ecosystem-level backstop.

**Cross-layer chain to watch:** L1 (document injection succeeds) → L3 (if the gate isn't truly external/deterministic, a persuasive output slips through) → L4 (payment API called) → L7 (real financial loss, regulatory exposure).

---

# 4. Coordinator Agent — MAESTRO Threat Model

### L1 — Foundation Models

| Threat | Type | Description |
|---|---|---|
| Reasoning-driven illegal transition | Agentic | LLM "decides" a state transition the formal state machine doesn't actually permit |
| Escalation suppression via reasoning | Agentic | Model under/over-weighs an ambiguous signal, failing to trigger a needed escalation |
| Summarization drift | Agentic | Multi-hop LLM summarization of sub-agent outputs loses or distorts material facts |

**Mitigations:** state transitions validated against a code-level transition table — LLM proposes, code enforces; deterministic escalation triggers run alongside LLM-assisted ones; case summaries pull directly from sub-agent structured outputs, not paraphrase-of-paraphrase.

### L2 — Data Operations

| Threat | Type | Description |
|---|---|---|
| State store as a new high-value target | Agentic | Journey state is a discrete, tamperable asset that didn't exist in a simpler design |
| In-place mutation hiding tampering | Traditional | A mutable "current status" field with no history makes falsified state hard to detect |
| Cross-agent data over-collection | Agentic | Coordinator passing more raw data to sub-agents than the minimum required |

**Mitigations:** event-sourced, append-only state store; write access restricted to the Coordinator's identity alone; Coordinator passes only minimum required parameters to each sub-agent call.

### L3 — Agent Frameworks

| Threat | Type | Description |
|---|---|---|
| Confused deputy | Agentic | Coordinator's invoke authority used (via manipulation) to call a sub-agent with attacker-influenced parameters |
| Over-broad tool/agent access | Agentic | Coordinator able to enumerate or call agents/tools beyond the three registered sub-agents |
| Configuration-modification authority | Agentic | Coordinator's identity able to modify a sub-agent's config/permissions rather than just invoke it |

**Mitigations:** each sub-agent independently re-validates whatever Coordinator hands it; Coordinator identity scoped to invoke-only, no configuration-modification rights; tool permissions scoped so Coordinator cannot call anything beyond the three registered sub-agents.

### L4 — Deployment and Infrastructure

| Threat | Type | Description |
|---|---|---|
| Overprivileged identity | Traditional | Coordinator granted broad access "because it touches everything," including direct payment/PHI access |
| Webhook/event replay | Traditional | Duplicate event delivery triggering duplicate sub-agent invocations |
| Direct database access bypassing Coordinator logic | Traditional | Standing write access to the state store outside the Coordinator's own transition logic |

**Mitigations:** verified absence of direct access to the payment API, risk-scoring API, or raw health/claim data — invoke sub-agents only; idempotency keys derived from `(applicant_id, event, attempt)`; webhook/event ingestion deduplicated before reaching Coordinator logic; state-store writes restricted to Coordinator's identity, alerted on any write from elsewhere.

### L5 — Evaluation and Observability

| Threat | Type | Description |
|---|---|---|
| Undetected stuck/abandoned journeys | Traditional | A journey silently stalls, mistaken for normal applicant delay |
| No end-to-end correlation | Traditional | Individual agent logs exist, but no single trace ties a journey's full lifecycle together |
| Escalation-event blending | Agentic | Escalation events logged together with routine state transitions, obscuring dismissal/suppression |

**Mitigations:** explicit timeout/staleness monitoring surfaced to ops; `journey_id`-correlated end-to-end tracing; escalation events logged as their own immutable stream, with dismissals requiring a logged reason.

### L6 — Security and Compliance

| Threat | Type | Description |
|---|---|---|
| Transition-table changes unreviewed | Traditional | Changes to legal state transitions shipped without the same review rigor as the disbursement gate |
| Ambiguous accountability across a multi-day journey | Traditional | Difficult to establish after the fact whether a delay or bypass was a bug or manipulation |

**Mitigations:** transition table versioned and change-reviewed like production code; full journey audit trail sufficient to reconstruct intent and timing on demand.

### L7 — Agent Ecosystem

| Threat | Type | Description |
|---|---|---|
| Privilege aggregation as a systemic risk | Agentic | Coordinator's orchestration breadth makes it the single point with the widest blast radius if compromised |
| Adversarial multi-day manipulation | Agentic | Attacker deliberately timing actions across a multi-day journey to exploit staleness/timeout gaps |
| Human-review fatigue as an exploitable pattern | Traditional | High escalation volume causing human underwriters to rubber-stamp reviews |

**Mitigations:** blast radius intentionally bounded by design (invoke-only, no direct payment/PHI access); adversarial state-machine testing targeting multi-day timing gaps; escalation volume monitored so review quality doesn't silently degrade under load.

**Cross-layer chain to watch:** L1 (LLM proposes an illegal or premature transition) → L3 (confused-deputy invocation of a sub-agent with bad parameters) → L2 (state store reflects a transition that shouldn't have happened) → L7 (a claim gets processed against a journey state that was never legitimately reached).

---

## System-level takeaway

Across all four agents, real-world damage in every identified cross-layer chain requires **failures at two or more layers**, not a single point of failure — an L1 model manipulation *and* a missing L3/L4 enforcement point that should have caught it. No agent's safety model relies on the LLM behaving correctly as the sole safeguard.

Use this as a standing test for any new agent added to the system: **for every plausible L1 failure, can you name the layer-2-or-later control that catches it?** If not, that's the next gap to close before the agent goes to production.

| Agent | Highest-leverage single control |
|---|---|
| Intake | Prompt injection defenses + independent output validation before persistence |
| Underwriting | Programmatic (non-LLM) enforcement that pricing values come verbatim from the API |
| Claims | Zero IAM permission to the payment API on the Claims Agent's own identity |
| Coordinator | Code-level state-transition validation (LLM proposes, code enforces) |
