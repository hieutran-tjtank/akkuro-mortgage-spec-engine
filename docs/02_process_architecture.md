# 02. Process Architecture & Workflow Modeling

**System:** Akkuro Mortgage Platform  
**Model:** BPMN 2.0 Origination Swimlanes & State Transition Matrix  
**Author:** Senior Business Analyst  

---

## 1. Process Overview

The Dutch mortgage origination lifecycle is an asynchronous, multi-actor pipeline spanning from initial borrower consultation to final notary deed execution and fund settlement.

```mermaid
sequenceDiagram
    autonumber
    actor B as Borrower
    actor A as Intermediary Advisor
    participant S as Akkuro Engine
    actor U as Lender Underwriter
    actor N as Civil-Law Notary

    B->>A: Provide personal & property details
    A->>S: Model scenarios & submit Application (DRAFT -> SUBMITTED)
    S-->>A: Generate document intake checklist
    A->>S: Upload & verify mandatory documents (DOCUMENT_VERIFICATION)
    A->>S: Promote complete dossier to Underwriting
    S->>U: Route dossier with automated compliance audit
    U->>S: Review & approve credit risk (CREDIT_ASSESSMENT)
    S-->>U: Generate Binding Offer (BINDING_OFFER_ISSUED)
    U->>B: Issue Binding Offer contract
    B->>S: Sign contract within 14-day window (ACCEPTED_BY_BORROWER)
    S->>N: Transmit Notary Closing Package (Passeeropdracht)
    N->>S: Confirm deed execution & fund release (NOTARY_PASSED)
```

---

## 2. BPMN 2.0 Swimlane Workflow

```mermaid
flowchart TD
    subgraph Borrower_Advisor["Borrower & Intermediary Advisor"]
        A1["Collect Financial Data & Preferences"] --> A2["Simulate LTV & Nibud Capacity"]
        A2 --> A3["Submit Application (DRAFT -> SUBMITTED)"]
        A3 --> A4["Upload Mandatory Dossier Documents"]
        A4 --> A5["Verify Income, Appraisal & Purchase Contract"]
        A5 --> A6["Submit Completed Dossier to Underwriting"]
    end

    subgraph Akkuro_Engine["Akkuro Rule Engine (Automated)"]
        E1{"Schema & Completeness Check"}
        E2{"Automated Rules Validation<br/>(LTV ≤ 100/106%, NHG, Nibud)"}
        E3["Lock Rate & Generate Offer Snapshot"]
        E4{"14-Day Cooling-off Timer"}
    end

    subgraph Underwriting["Lender Underwriter (Acceptant)"]
        U1["Review Dossier Discrepancies"]
        U2{"Credit Risk Decision"}
        U3["Issue Formal Rejection"]
        U4["Issue Binding Offer (Definitieve Offerte)"]
    end

    subgraph Notary["Civil-Law Notary (Notaris)"]
        N1["Receive Closing Instructions"]
        N2["Draft Mortgage Deed (Hypotheekakte)"]
        N3["Execute Deed & Disburse Funds (NOTARY_PASSED)"]
    end

    A3 --> E1
    E1 -- Pass --> A4
    A6 --> E2
    E2 -- Compliant --> U1
    E2 -- Violation --> U1
    U1 --> U2
    U2 -- Reject --> U3
    U2 -- Approve --> U4
    U4 --> E3
    E3 --> E4
    E4 -- Signed within 14 days --> N1
    E4 -- Expired --> A1
    N1 --> N2
    N2 --> N3
```

---

## 3. State Transition Matrix

The table below defines every legal lifecycle status transition in the Akkuro platform. Transitions outside this matrix are rejected by the engine.

| From State | To State | Trigger Event | Allowed Actor(s) | Guard Condition / Validation Gate |
|---|---|---|---|---|
| `DRAFT` | `SUBMITTED` | Submit Application | Borrower, Advisor | All mandatory applicant, income, and property fields populated. |
| `SUBMITTED` | `DOCUMENT_VERIFICATION` | Intake Routing | Advisor, System | Automated schema and initial anti-fraud checks passed. |
| `DOCUMENT_VERIFICATION` | `CREDIT_ASSESSMENT` | Dossier Verified | Advisor | 4 mandatory documents (Employer, UWV, NWWI Appraisal, Purchase Agreement) marked Verified. |
| `CREDIT_ASSESSMENT` | `BINDING_OFFER_ISSUED` | Underwriting Approval | Underwriter | LTV $\le$ cap, requested loan $\le$ Nibud capacity, NHG rules validated. |
| `CREDIT_ASSESSMENT` | `REJECTED` | Underwriting Rejection | Underwriter | Statutory violation or credit risk threshold breached. Reason logged. |
| `BINDING_OFFER_ISSUED` | `ACCEPTED_BY_BORROWER` | Offer Acceptance | Borrower | Digital signature completed within 14-day statutory reflection window. |
| `BINDING_OFFER_ISSUED` | `EXPIRED` | Reflection Timeout | System | Statutory 14-day window lapsed without borrower signature (`BR-OFFER-02`). |
| `ACCEPTED_BY_BORROWER` | `NOTARY_PASSED` | Deed Execution | Notary | Mortgage deed signed and escrow fund release confirmed. |
| *Any Pre-Closing State* | `CANCELLED` | Withdrawal | Borrower, Advisor | Application formally withdrawn by customer prior to deed execution. |

---

## 4. Role-Based Access Control (RBAC) & Four-Eyes Principle

To prevent fraud and maintain regulatory audit trails, critical origination stages enforce strict separation of duties:

| Action | Borrower (*Klant*) | Advisor (*Adviseur*) | Underwriter (*Acceptant*) | Notary (*Notaris*) |
|---|:---:|:---:|:---:|:---:|
| Initiate Application | ✅ Full | ✅ Full | ❌ None | ❌ None |
| Upload & Verify Documents | ⚠️ Upload only | ✅ Verify & Sign-off | 👁️ Audit only | ❌ None |
| Configure Loan Scenarios | 👁️ View quote | ✅ Configure & Model | 👁️ Review | ❌ None |
| Issue Binding Loan Offer | ❌ Forbidden | ❌ Forbidden | ✅ Exclusive | ❌ None |
| Execute Mortgage Deed | ❌ Forbidden | ❌ Forbidden | ❌ Forbidden | ✅ Exclusive |
