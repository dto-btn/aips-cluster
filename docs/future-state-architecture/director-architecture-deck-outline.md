# Director Architecture Deck — Slide Outline

**Presentation Title:** Future-State AI Platform Architecture  
**Audience:** Director / Senior Leadership  
**Purpose:** Illustrate the target architecture, explain the rationale for the proposed hosting changes, and present the sovereignty posture for review and decision.

> **How to use this outline:** Each slide section below is a direct starting point for a PowerPoint slide. Speaker notes and prompts are marked with 📝. Replace all `[PLACEHOLDER]` items with confirmed values before presenting.

---

## Slide 1 — Title

**Title:** Future-State AI Platform Architecture  
**Subtitle:** Stability · Scalability · Sovereignty Posture  
**Date:** [DATE]  
**Presented by:** [PRESENTER NAME / TEAM]

📝 *Speaker note:* Set context — this is a high-level architectural direction for review and discussion. Details and commitments will be confirmed through further assessment and procurement.

---

## Slide 2 — Executive Summary

**Key Points:**
- The current CANChat platform runs on ThinkOn/TKO infrastructure (Canadian-owned, Canadian-operated).
- The proposed future state moves the **core application platform** to Azure (target: Canadian region — subject to validation), while **retaining ThinkOn/TKO for Cohere North and approved Cohere LLMs** (Command A and future models).
- Two primary architectural differentiators drive this recommendation:
  1. **Stability** — improved platform resilience, managed services SLAs, and HA/DR capabilities.
  2. **Scalability** — elastic, on-demand compute and storage aligned to variable workload demand.
- Microsoft Entra ID (Azure AD) remains the identity provider regardless of workload placement.
- A sovereignty posture assessment is underway; Canadian deployment and provider corporate jurisdiction are tracked independently for each service.

📝 *Speaker note:* Emphasize that this is a direction for approval-in-principle. Target regions, HA objectives, vendor commitments, and sovereignty assessments require validation before finalization.

---

## Slide 3 — Current State Summary

**Title:** Where We Are Today

| Element | Current State |
|---|---|
| **Core platform hosting** | ThinkOn/TKO (Canadian-owned, Canadian-operated) |
| **Identity provider** | Microsoft Entra ID (Azure AD) |
| **AI / LLM models** | Cohere Command A, Azure OpenAI (via LiteLLM gateway), others |
| **Deployment model** | Kubernetes on ThinkOn VCF |
| **Data services** | PostgreSQL, Redis, object storage, Qdrant (all on ThinkOn/TKO) |
| **Sovereignty posture** | Core platform: CA / CA (target) · Identity: subject to per-service assessment · External AI: TBC |

**Key Limitations:**
- Scaling requires provisioning additional physical or virtual capacity within TKO — lead times and capacity ceilings apply.
- Managed service SLAs and HA/DR options are constrained relative to major public cloud providers.
- Operational burden for platform management (patching, upgrades, DR testing) sits with the team.

📝 *Speaker note:* This slide is a summary — do not list every component. Focus on what changes and why.

---

## Slide 4 — Future-State Architecture Diagram

**Title:** Future-State Architecture — Service Boundaries and Sovereignty Posture

> *(Insert rendered diagram from `future-state-architecture.mmd` here.)*

**Diagram key messages to call out verbally:**
1. Users access the platform through a secure perimeter (DNS, WAF, ingress).
2. Microsoft Entra ID remains the single identity provider across all workloads.
3. The **core application platform** (application services, data, AI gateway, operations) moves to Azure (target: Canadian region — subject to validation).
4. ThinkOn/TKO hosts only the **Cohere North platform and Cohere LLMs** (CA / CA).
5. The **AI gateway** is the single controlled integration point to all model providers — no application components call models directly.
6. External AI providers (Azure OpenAI and others) are behind the gateway and assessed individually.

📝 *Speaker note:* Refer to the sovereignty badges on each zone. Stress that "Canada-hosted" does not automatically mean fully sovereign — each service is assessed on two independent dimensions. [PLACEHOLDER: confirm target Azure region(s) before presenting.]

---

## Slide 5 — Rationale: Stability

**Title:** Why Azure for Core Platform? — Stability

**Current pain points addressed:**
- Single-cluster failure modes in the current TKO deployment → Azure managed Kubernetes (AKS) provides multi-zone and multi-region options.
- Dependency on physical TKO capacity for HA — Azure provides SLA-backed availability zones [PLACEHOLDER: target SLA — subject to validation and service tier selection].
- Database and storage resilience relies on team-managed backup procedures → Azure offers managed backup, geo-redundant storage, and zone-redundant databases as platform capabilities.

**Future-state stability improvements:**
- Managed Kubernetes control plane with vendor-SLA-backed availability [PLACEHOLDER: target uptime SLA — confirm with Azure service tier selection].
- Zone-redundant or geo-redundant data services where applicable (subject to data-residency validation per service).
- Managed upgrade and patching cadence reduces operational risk.
- Integrated platform health monitoring and alerting.
- DR runbook and recovery-time objectives to be defined [PLACEHOLDER: RTO / RPO targets — confirm with business and operations teams].

📝 *Speaker note:* Do not state specific SLA numbers unless they have been verified against the selected service tiers and committed contracts. Use "target" language throughout.

---

## Slide 6 — Rationale: Scalability

**Title:** Why Azure for Core Platform? — Scalability

**Current limitations:**
- TKO infrastructure scaling requires manual capacity requests and physical provisioning lead times.
- Horizontal pod autoscaling is constrained by available node capacity in the cluster.
- Storage expansion and compute bursting are bounded by the TKO environment's physical limits.

**Future-state scalability improvements:**
- Azure AKS node pool autoscaling — compute scales automatically to match workload demand.
- Serverless/consumption-tier options for background processing and document ingestion workloads reduce idle cost.
- Azure Blob / managed storage scales without pre-provisioning.
- AI gateway (LiteLLM or equivalent) can route traffic across multiple model providers to distribute load.
- Cohere North on ThinkOn/TKO scales through the Cohere platform; Azure serves as the fallback and alternative model path.
- Cost model shifts from fixed infrastructure toward consumption-aligned spend [PLACEHOLDER: cost modelling and financial approval required].

📝 *Speaker note:* Highlight the decoupling of TKO from the main application scalability path. TKO/Cohere North remains in the picture for specific model capability — it is not the bottleneck for application scaling.

---

## Slide 7 — Sovereignty Model

**Title:** Sovereignty Posture — Two Independent Dimensions

**Framework:**

| Dimension | Question | Label |
|---|---|---|
| **Deployment / Data Jurisdiction** | Where is the workload, data, backups, and support located and governed? | CA = Canada (target / validated) · TBC = not yet confirmed |
| **Provider Corporate Jurisdiction** | Is the provider Canadian-owned, Canadian-operated, and subject only to Canadian law? | CA = Canadian-controlled · Foreign = foreign-controlled |

**Why both dimensions matter:**

> *"Canada-hosted" does not mean "fully sovereign."*  
> A workload in an Azure Canadian region is data-resident in Canada (for services and configurations where this has been validated), but Microsoft is a foreign-controlled provider subject to U.S. jurisdiction. These are distinct risk considerations and should not be conflated.

**Labels used in this architecture:**

| Label | Meaning |
|---|---|
| **CA / CA** | Canadian deployment (target/validated) + Canadian-controlled provider |
| **CA / Foreign** | Canadian deployment (target/validated) + Foreign-controlled provider |
| **TBC / Foreign** | Deployment jurisdiction not yet selected or verified + Foreign-controlled provider |
| **TBC / TBC** | Both dimensions not yet assessed |

📝 *Speaker note:* This model is designed to be defensible and honest. Avoid using the word "sovereign" as a blanket term — use the two-part label so stakeholders understand exactly what has and has not been verified.

---

## Slide 8 — Sovereignty Posture Table

**Title:** Service-by-Service Sovereignty Posture (Summary)

| Zone / Service | Deployment / Data Jurisdiction | Provider Corporate Jurisdiction | Label | Notes |
|---|---|---|---|---|
| ThinkOn/TKO — Cohere North platform | CA (Canadian-operated) | CA (Canadian-controlled) | **CA / CA** | Subject to contract and subcontractor validation |
| Cohere LLMs (Command A, future models) | CA (hosted via Cohere North) | TBC (Cohere Inc. — assess separately) | **CA / TBC** | Model vendor jurisdiction assessed independently from hosting environment |
| Azure — Core Application Platform | CA (target: [PLACEHOLDER: region]) | Foreign (Microsoft — U.S.-headquartered) | **CA / Foreign** | Per-service data-residency validation required; not all Azure services guarantee Canadian residency by default |
| Microsoft Entra ID (Azure AD) | TBC (identity-service data residency requires per-tenant validation) | Foreign (Microsoft) | **TBC / Foreign** | Validate identity-tenant data-residency configuration and log/telemetry location separately |
| Azure AI / Azure OpenAI | CA (target where Canadian region selected and validated) | Foreign (Microsoft) | **CA / Foreign** (where verified) · **TBC / Foreign** otherwise | Per-service validation required; not assumed from application region |
| Other approved cloud / AI providers | TBC | Foreign | **TBC / Foreign** | Each provider requires independent assessment before use |

📝 *Speaker note:* This table will be updated as assessments are completed. Reference `sovereignty-assessment-template.md` for the per-service evidence and validation process.

---

## Slide 9 — Decisions and Next Steps

**Title:** Decisions Required & Next Steps

**Decisions for leadership:**
1. Approve the proposed future-state architectural direction (core platform to Azure, Cohere North retained on TKO).
2. Confirm acceptable sovereignty posture for Azure-hosted components (CA / Foreign classification).
3. Authorize sovereignty assessment process for each service/vendor.

**Next steps:**

| Action | Owner | Target |
|---|---|---|
| Confirm target Azure region(s) for core platform | [PLACEHOLDER] | [DATE] |
| Complete per-service sovereignty assessment (use template) | [PLACEHOLDER] | [DATE] |
| Validate Entra ID data-residency configuration | [PLACEHOLDER] | [DATE] |
| Define and confirm RTO / RPO targets for DR | [PLACEHOLDER] | [DATE] |
| Financial approval and cost modelling for Azure workloads | [PLACEHOLDER] | [DATE] |
| Cohere Inc. corporate jurisdiction and subcontractor assessment | [PLACEHOLDER] | [DATE] |
| Prepare detailed design for core platform migration | [PLACEHOLDER] | [DATE] |

📝 *Speaker note:* This slide is for follow-up action — assign owners before the meeting ends. All PLACEHOLDER items are genuine open questions and should not be assumed or pre-answered during presentation.

---

## Slide 10 — Appendix / Reference (Optional)

**Title:** Reference Materials

- Architecture diagram source: `docs/future-state-architecture/future-state-architecture.mmd`
- Sovereignty assessment template: `docs/future-state-architecture/sovereignty-assessment-template.md`
- Current-state network-flow diagram: *(link to existing repo diagram)*
- Microsoft data residency reference: [azure.microsoft.com — Data Residency](https://azure.microsoft.com/en-ca/explore/global-infrastructure/data-residency/)
- ThinkOn sovereign cloud: [thinkon.com/sovereign-cloud](https://thinkon.com/sovereign-cloud/)

📝 *Speaker note:* Appendix slide — include or remove based on presentation format and time.
