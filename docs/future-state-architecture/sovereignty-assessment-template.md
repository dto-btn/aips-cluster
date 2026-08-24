# Sovereignty Assessment Template

**Purpose:** Capture and track the independent sovereignty assessment for each service or vendor component in the future-state architecture. Complete one copy of this table per service. Maintain completed assessments as records supporting architecture decisions.

> **Important:** Canadian data residency and Canadian provider sovereignty are two distinct and independent dimensions. Do not assume that a service is fully sovereign because it is deployed in a Canadian region. Each dimension requires documented evidence and contract validation.

---

## Sovereignty Label Definitions

| Label | Deployment / Data Jurisdiction | Provider Corporate Jurisdiction | Meaning |
|---|---|---|---|
| **CA / CA** | Canada (validated) | Canadian-controlled (validated) | Canadian deployment confirmed AND provider is Canadian-owned/operated under Canadian law |
| **CA / Foreign** | Canada (validated) | Foreign-controlled | Workload/data in Canada (validated per service) AND provider is subject to foreign jurisdiction |
| **TBC / Foreign** | Not yet confirmed | Foreign-controlled | Deployment jurisdiction not yet selected or validated; provider is known to be foreign-controlled |
| **TBC / TBC** | Not yet confirmed | Not yet assessed | Both dimensions require assessment; use as initial placeholder only |

> ⚠ Never use "sovereign" as an unqualified term. Always specify both dimensions separately.

---

## Assessment Record

### 1. Service / Component Identification

| Field | Value |
|---|---|
| **Service / Component Name** | _(e.g., PostgreSQL on Azure, Cohere Command A via Cohere North, Azure Blob Storage)_ |
| **Purpose / Function** | _(What does this service do in the architecture?)_ |
| **Architecture Zone** | _(e.g., Azure Core Platform · ThinkOn/TKO · External Provider)_ |
| **Assessment Date** | |
| **Assessed by** | |
| **Assessment Status** | `Draft` / `In Review` / `Validated` / `Accepted with Residual Risk` |

---

### 2. Deployment and Data Jurisdiction

| Field | Value |
|---|---|
| **Primary deployment region / location** | _(e.g., Canada Central, Canada East, ThinkOn Toronto, Unknown — TBC)_ |
| **Data-at-rest location** | _(Where is data stored? Is this the same as the deployment region? Document evidence.)_ |
| **Data-in-transit path** | _(Does data transit through regions outside Canada? Describe routing.)_ |
| **Backup / DR location** | _(Where are backups stored? Where is the DR failover site? Are these in Canada?)_ |
| **Log and telemetry location** | _(Where are operational logs, audit logs, and telemetry sent and retained?)_ |
| **Support access jurisdiction** | _(From which locations can vendor support staff access the environment/data?)_ |
| **Data-residency commitment** | _(Is there a documented, contractual data-residency commitment? Cite the reference.)_ |
| **Deployment jurisdiction label** | `CA` / `Foreign` / `TBC` |
| **Evidence / Reference** | _(Link to contract, service terms, vendor documentation, or internal assessment)_ |

---

### 3. Provider Corporate Jurisdiction

| Field | Value |
|---|---|
| **Provider / Vendor name** | |
| **Provider headquarters / jurisdiction** | _(Country and legal jurisdiction of the corporate entity)_ |
| **Applicable foreign law exposure** | _(e.g., U.S. CLOUD Act, FISA, other foreign-state access obligations)_ |
| **Canadian subsidiary / entity** | _(Is there a Canadian legal entity? Does it control the service or is it a reseller/agent?)_ |
| **Control-plane jurisdiction** | _(Where is the control plane operated? Who has administrative/privileged access?)_ |
| **Subcontractors / sub-processors** | _(List known subcontractors and their jurisdictions. Note if a complete list is unavailable.)_ |
| **Provider jurisdiction label** | `CA` (Canadian-controlled) / `Foreign` / `TBC` |
| **Evidence / Reference** | _(Link to contract, data processing agreement, privacy statement, or vendor disclosure)_ |

---

### 4. Data Classification and Sensitivity

| Field | Value |
|---|---|
| **Data classifications processed** | _(e.g., Unclassified, Protected A, Protected B — specify per applicable policy)_ |
| **Personally Identifiable Information (PII)** | `Yes` / `No` / `TBC` — describe if yes |
| **Data subject to specific legislation** | _(e.g., Privacy Act, PIPEDA, ATIP, Treasury Board directives — list applicable ones)_ |
| **Maximum data classification permitted** | _(Based on current assessment, what is the maximum classification suitable for this service? State basis.)_ |

---

### 5. Security and Technical Controls

| Field | Value |
|---|---|
| **Encryption at rest** | _(Standard, algorithm, key management location — is the key managed by GC or provider?)_ |
| **Encryption in transit** | _(Protocol version, certificate authority, mutual TLS if applicable)_ |
| **Key management jurisdiction** | _(Where are encryption keys stored and managed? Who can access them?)_ |
| **Network connectivity model** | _(e.g., Private endpoint, VNet integration, public internet with TLS, ExpressRoute — describe)_ |
| **Access control model** | _(How is privileged access controlled and audited? MFA? PAM? Just-in-time?)_ |
| **Audit logging** | _(Are all access and change events logged? Where are logs stored? Retention period?)_ |

---

### 6. Contract and Compliance References

| Field | Value |
|---|---|
| **Contract / agreement reference** | _(Contract name, number, or internal reference)_ |
| **Data Processing Agreement (DPA) in place** | `Yes` / `No` / `In Progress` |
| **Relevant certifications** | _(e.g., ISO 27001, SOC 2 Type II, FedRAMP, ITSG-33 profile — list with dates)_ |
| **GC security assessment / ATO** | _(Has a GC Authority to Operate or equivalent been granted? Reference.)_ |
| **Privacy impact assessment (PIA)** | `Complete` / `In Progress` / `Required` / `Not Required` |
| **Outstanding contractual gaps** | _(List any missing clauses, pending DPA terms, or unresolved contract issues)_ |

---

### 7. Sovereignty Assessment Summary

| Field | Value |
|---|---|
| **Deployment / Data Jurisdiction label** | `CA` / `Foreign` / `TBC` |
| **Provider Corporate Jurisdiction label** | `CA` / `Foreign` / `TBC` |
| **Combined sovereignty label** | _(e.g., CA / CA · CA / Foreign · TBC / Foreign · TBC / TBC)_ |
| **Label confidence** | `Confirmed` (evidence in hand) / `Target` (planned but not yet validated) / `Assumed` (not yet evidenced — requires upgrade to Confirmed before production) |

---

### 8. Residual Risks and Mitigations

| Risk | Likelihood | Impact | Mitigation | Residual Risk Owner |
|---|---|---|---|---|
| _(e.g., Backup data exits Canada)_ | | | | |
| _(e.g., Vendor support access from foreign jurisdiction)_ | | | | |
| _(e.g., Control plane telemetry sent to foreign region)_ | | | | |
| _(Add rows as required)_ | | | | |

---

### 9. Architecture Decision

| Field | Value |
|---|---|
| **Recommendation** | `Approve for use` / `Approve with conditions` / `Reject` / `Defer pending further assessment` |
| **Conditions / requirements** | _(List any conditions that must be met before or after approval)_ |
| **Decision maker** | |
| **Decision date** | |
| **Review date** | _(When should this assessment be re-validated? e.g., at contract renewal, at next major architecture review)_ |
| **Notes** | |

---

## Completed Assessments Index

Maintain a list of completed assessments here, or link to a tracker:

| Service / Component | Label | Status | Assessment Date | Location of Record |
|---|---|---|---|---|
| ThinkOn/TKO — Cohere North platform | CA / CA (target) | Target — requires validation | [DATE] | [LINK] |
| Azure Core Platform (AKS, App Services, etc.) | CA / Foreign (target) | Target — per-service validation required | [DATE] | [LINK] |
| Microsoft Entra ID (Azure AD) | TBC / Foreign | In progress | [DATE] | [LINK] |
| Azure AI / Azure OpenAI | TBC / Foreign | Not started | | |
| Cohere Command A (model vendor) | TBC / TBC | Not started | | |
| Other providers | TBC / TBC | Not started | | |

---

*This template is subject to revision as assessment methodology and GC policy guidance evolves. All assessments are working documents until marked `Validated` and signed off.*
