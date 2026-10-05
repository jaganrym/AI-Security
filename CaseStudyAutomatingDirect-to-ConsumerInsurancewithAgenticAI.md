# Case Study: Automating Direct-to-Consumer Insurance with Agentic AI

## Executive Summary
**JRT Life**, an 80-person direct-to-consumer life and health insurance startup, successfully replaced its slow, manual pipeline with a coordinated four-agent AI system. Facing fierce competition from legacy insurers, the company leveraged agentic architecture to dramatically accelerate application processing and claims fulfillment while maintaining strict security boundaries over sensitive financial and health data.

## 🏢 Company Profile
* **Company Name:** JRT Life
* **Company Size:** 80 employees
* **Industry:** Direct-to-Consumer (DTC) Life & Health Insurance
* **Core Challenge:** Competing with legacy insurers on speed and operational efficiency.

## 🤖 System Architecture & Agent Roles
JRT Life deployed a specialized, multi-agent AI system where each agent is ring-fenced with distinct capabilities, tools, and data access permissions.

| Agent | Job / Core Responsibilities | Touches & Tool Access |
| :--- | :--- | :--- |
| **Intake Agent** | • Chats with applicants via natural conversation<br>• Extracts health history and lifestyle data from free-text answers | • Sensitive health data (PII/PHI-equivalent)<br>• Data extraction tools |
| **Underwriting Agent** | • Processes extracted applicant data into risk profiles<br>• Calculates and sets specific policy prices | • Internal risk-scoring API<br>• Proprietary pricing engines |
| **Claims Agent** | • Reads and parses submitted claim documents<br>• Decides claim validity and legitimacy | • Payment gateway API<br>• Corporate financial exposure controls |
| **Coordinator Agent** | • Orchestrates the workflows of the other three agents<br>• Tracks system state across multi-day user journeys<br>• Escalates complex edge cases to human underwriters | • All of the above data streams<br>• Cross-agent communication pipelines<br>• Human-in-the-loop escalation paths |

## 🔄 Workflow Operations
<img width="1145" height="920" alt="image" src="https://github.com/user-attachments/assets/287329c0-23ac-47ed-a40b-02419a8f1332" />


## 🔄 Multi-Agent System Workflow Description
The diagram represents the hub-and-spoke operational flow managed by **JRT Life's** core AI system:

* **Central Control:** The **Coordinator Agent** acts as the central brain and state manager. Every external input or internal handoff flows directly through it.
* **Human Oversight:** When an anomaly or high-risk metric is identified, the **Coordinator Agent** instantly routes the workflow to a **Human Underwriter** for manual verification.
* **Specialised Execution:** The three execution agents (**Intake**, **Underwriting**, and **Claims**) run within isolated environments, communicating only with the Coordinator to ensure clean data boundaries and secure API usage.

### The Onboarding Journey
1. The **Intake Agent** collects conversational health profiles.
2. The **Coordinator Agent** sanitizes the extracted profiles and passes relevant data points to the next stage.
3. The **Underwriting Agent** processes these metrics against risk engines to return a live policy price.

### The Claims Journey
1. The **Coordinator Agent** receives a claims filing and triggers document ingestion.
2. The **Claims Agent** parses the documents, determines validity, and initiates the payout sequence through the external payment gateway.
3. If an anomaly is detected at any point, the **Coordinator Agent** halts the pipeline and alerts a human underwriter.

## 📈 Business Benefits
* **Unprecedented Speed:** Transitions the company from multi-day manual underwriting cycles to near-instantaneous policy issuance.
* **Operational Leverage:** Allows an 80-person startup to scale processing volume efficiently without proportionally increasing head count.
* **Risk Mitigation:** Enforces rigid compliance and financial guardrails, ensuring that payment gateways and PHI databases are only accessible by specialized, highly audited agent nodes.

***
*Created with OneNote.*
