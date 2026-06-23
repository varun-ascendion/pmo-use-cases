# BRD: Payments Middleware Platform – Enforce Structured/Hybrid Beneficiary Address on ISO 20022 Messages (ISO 20022 Structured Address Mandate)

| Field | Value |
|---|---|
| BRD Reference | BRD-PAYMENTS-2026-002 |
| Regulation | ISO 20022 Mandate on rejecting payments messages with unstructured beneficiary address and to allow only Structured or Hybrid address formats (governing body: To be confirmed — typically SWIFT CBPR+ / Payment Market Infrastructures under ISO 20022 migration) [R1] |
| Change Type | Rule-engine update + Schema validation enhancement + Message rejection workflow |
| Status | Draft v0.1 |
| Date | 23 June 2026 |

## 1. Purpose
This BRD defines the changes required in the Payments Middleware Platform to enforce the ISO 20022 mandate that rejects payment messages carrying an **unstructured** beneficiary postal address and accepts only **Structured** or **Hybrid** address formats [R1]. The change introduces new pre-processing validation rules and message rejection behaviour at the inbound ISO 20022 ingestion path of the middleware [A1]. Explicitly excluded: changes to ordering customer (debtor) address handling unless mandated by the same rulebook, and any sanctions/AML re-screening logic [R1].

## 2. Regulatory Driver
The ISO 20022 mandate requires that all in-scope payment messages (for example pacs.008, pacs.009, pain.001) carry the beneficiary (Creditor) postal address in **Structured** format (discrete elements such as `StreetName`, `BuildingNumber`, `PostCode`, `TownName`, `Country`) or **Hybrid** format (a combination of structured elements plus a limited address line), and that messages containing fully **Unstructured** addresses (free-text `AddressLine` only) must be rejected [R1]. Effective date: **To be confirmed** — flagged in Section 7 as an assumption pending publication of the formal cutover date by the governing scheme [R1].

**Gap:** The retrieved Payments Middleware architecture currently accepts ISO 20022 inbound messages without enforcing the Structured/Hybrid format constraint on the Creditor `PostalAddress` block, and does not have a dedicated validation rule or rejection reason code for unstructured beneficiary addresses [A1]. Consequence of inaction: non-compliant messages would be forwarded to clearing systems that reject them, causing payment failures, SLA breaches, and regulatory censure [A1].

## 3. Functional Requirements

| ID | Requirement | Citations |
|---|---|---|
| FR-01 | The middleware must inspect the Creditor `PostalAddress` element on every inbound ISO 20022 payment message in scope and classify it as Structured, Hybrid, or Unstructured. | [R1][A1] |
| FR-02 | The middleware must reject any in-scope message whose Creditor `PostalAddress` is classified as Unstructured, prior to forwarding to downstream clearing. | [R1][A1] |
| FR-03 | The middleware must return a deterministic rejection reason code and human-readable description identifying the address-format failure to the originating channel. | [R1][A1] |
| FR-04 | The middleware must log every address-format validation outcome (pass/fail with classification) to the audit store with full message reference and timestamp. | [R1][A1] |
| FR-05 | The validation must be configurable per message type (pacs.008, pacs.009, pain.001, and any other in-scope ISO 20022 messages) and per effective date. | [R1][A1] |
| FR-06 | The middleware must continue to accept Hybrid addresses (structured elements combined with permitted address lines) without rejection. | [R1][A1] |
| FR-07 | A monitoring metric (count of rejected messages by reason "Unstructured Beneficiary Address") must be exposed to the operations dashboard. | [R1][A1] |

## 4. Application Impacts

| Component | Change Required | Dependencies | Citations |
|---|---|---|---|
| Inbound ISO 20022 message ingestion / parser | Extend parsing to extract Creditor `PostalAddress` sub-elements and feed the classification engine. | Schema registry; channel adapters | [A1][R1] |
| Validation / Rule Engine | Add new validation rule (VAL-01) and rule-engine update to classify address format and reject Unstructured. | Rule-engine versioning; sandbox testing | [A1][R1] |
| Rejection / Negative-response handler | Add new rejection reason code for unstructured beneficiary address; build response message to originator. | Reason-code catalogue; channel adapters | [A1][R1] |
| Audit & Logging service | Capture address classification result and rejection reason in the audit log for each message. | Audit log retention policy | [A1][R1] |
| Operations / Monitoring dashboard | Add rejection-volume metric and alerting threshold for unstructured-address rejections. | Metrics pipeline | [A1][R1] |
| Configuration / Reference Data | Maintain effective-date and per-message-type enable flags for the new rule. | Change-management workflow | [A1][R1] |

## 5. Data Model & Rule Changes

### 5.1 Entity Changes

| Entity | Field | Current | Target | Change | Citations |
|---|---|---|---|---|---|
| Payment Message (Creditor block) | `address_format_classification` | Not present | ENUM { STRUCTURED, HYBRID, UNSTRUCTURED } | Add derived field populated by validation engine | [A1][R1] |
| Rejection Reason Catalogue | `reason_code` | Existing set without an unstructured-address code | Add new code (e.g. `ADDR_UNSTRUCT_REJECT`) with description | Add new entry | [A1][R1] |
| Audit Log Record | `address_validation_result` | Not captured | Captured per message (classification + pass/fail) | Add field | [A1][R1] |

### 5.2 Rule Engine Logic

**Rule ID: PAY-BRE-ADDR-001 — Beneficiary Address Format Enforcement**

Current:
```
// No dedicated rule; Creditor.PostalAddress accepted in any format
accept(message)
```

Target:
```
classification = classifyAddress(message.Creditor.PostalAddress)
// classifyAddress returns STRUCTURED if structured sub-elements present and no free-text AddressLine,
// HYBRID if both structured sub-elements and permitted AddressLine present,
// UNSTRUCTURED if only free-text AddressLine present.

IF message.type IN (in_scope_message_types)
   AND effective_date <= today
   AND classification == UNSTRUCTURED
THEN
   reject(message, reason_code = "ADDR_UNSTRUCT_REJECT",
                   description = "Beneficiary address must be Structured or Hybrid per ISO 20022 mandate")
ELSE
   accept(message)
ENDIF
```
Citations: [R1][A1]

### 5.3 New Validation Rules

| Rule ID | Field | Rule | Error Behaviour | Citations |
|---|---|---|---|---|
| VAL-01 | Creditor `PostalAddress` | Must classify as STRUCTURED or HYBRID for in-scope ISO 20022 messages | Reject message; return reason code `ADDR_UNSTRUCT_REJECT`; log to audit | [R1][A1] |
| VAL-02 | Creditor `PostalAddress.Country` | Mandatory when classification is STRUCTURED or HYBRID | Reject message; return reason code `ADDR_COUNTRY_MISSING`; log to audit | [R1][A1] |

## 6. Out of Scope
- Debtor (ordering customer) postal address validation — reason: not explicitly covered by the supplied regulatory instruction [R1].
- Sanctions screening rule changes — reason: orthogonal control, not affected by address-format mandate [R1].
- Migration of legacy MT messages — reason: instruction is specific to ISO 20022 messages [R1].
- Changes to downstream clearing connectivity protocols — reason: format enforcement is performed at middleware ingress only [A1].

## 7. Assumptions & Constraints

| Ref | Assumption | Risk if Invalidated | Citations |
|---|---|---|---|
| A1 | The effective date of the mandate is the date currently published by the governing scheme; precise date is **To be confirmed**. | Late or early activation could cause non-compliance or premature rejections impacting customers. | [R1] |
| A2 | In-scope ISO 20022 message types are limited to customer credit transfers and financial institution credit transfers as handled by the middleware (e.g. pacs.008, pacs.009, pain.001); exact list **To be confirmed**. | Missing a message type could allow non-compliant messages to flow; including out-of-scope types could cause spurious rejections. | [R1][A1] |
| A3 | The definition of "Hybrid" follows the prevailing ISO 20022 usage guideline (structured elements plus a limited number of permitted address lines). | Misinterpretation could lead to incorrect classification and false rejections or false acceptances. | [R1] |
| A4 | The middleware's existing rule-engine framework supports adding new validation rules and rejection reason codes without architectural change. | If unsupported, a larger platform change would be required, extending timelines and cost. | [A1] |
| A5 | Downstream clearing systems will not require further address transformation once the middleware enforces Structured/Hybrid format. | Additional downstream changes may be needed, increasing scope. | [A1][R1] |

## 8. Acceptance Criteria
- A pacs.008 message with only free-text `AddressLine` in Creditor `PostalAddress` is rejected with reason code `ADDR_UNSTRUCT_REJECT`. [R1][A1]
- A pacs.008 message with fully Structured Creditor `PostalAddress` (e.g. `StreetName`, `BuildingNumber`, `PostCode`, `TownName`, `Country`) is accepted and forwarded. [R1][A1]
- A pacs.008 message with Hybrid Creditor `PostalAddress` (structured elements plus permitted address line) is accepted and forwarded. [R1][A1]
- Every validation outcome (pass/fail with classification) is written to the audit log with message reference and timestamp. [R1][A1]
- The operations dashboard displays a count of messages rejected due to unstructured beneficiary address. [R1][A1]
- Activation of the rule is controllable by effective date and message-type configuration. [R1][A1]
- No regression in processing of out-of-scope message types. [A1]

## 9. References

**Regulatory `[R#]`**

| Marker | Source (regulation, article/RTS/clause) | Verbatim excerpt |
|---|---|---|
| [R1] | ISO 20022 mandate (governing body and exact clause: To be confirmed; aligned with ISO 20022 migration rulebooks) | "ISO20022 Mandate on rejecting payments messages with unstructured beneficiary address and to allow only Structured or Hybrid address formats" |

**Architecture `[A#]`**

| Marker | Source location (section, component, rule ID) | Verbatim excerpt or paraphrase |
|---|---|---|
| [A1] | Retrieved document: `L1 change impact analyser/PACE/MARKDOWN-Reference Architecture - Payments Middleware.md` (varun-ascendion/pmo-use-cases, branch main) — Payments Middleware platform reference architecture (ingestion, validation/rule engine, rejection handling, audit/logging, operations monitoring components). | Paraphrase: The Payments Middleware reference architecture defines an inbound ISO 20022 ingestion path, a validation/rule-engine layer, a rejection-response handler with reason codes, an audit/logging service, and operational dashboards — the components impacted by this change. |

---
## Execution Summary
Feature addressed: enforcement of Structured/Hybrid Creditor postal address on inbound ISO 20022 payment messages in the Payments Middleware. Regulation implemented: ISO 20022 mandate rejecting unstructured beneficiary addresses. Headline impacts: new validation rule (VAL-01) and rule-engine update (PAY-BRE-ADDR-001), a new rejection reason code, audit-log enrichment, and an operations dashboard metric — all configurable by message type and effective date.

<!-- agent_id: change-impact-analyser | version: 2.3.0 | execution_id: exec-3c7e1a9d-5b2f-4a8e-9d6c-1f8a2b3c4d5e -->
