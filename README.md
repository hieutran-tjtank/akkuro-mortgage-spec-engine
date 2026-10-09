# Akkuro Mortgage Specification & Domain Repository

A dedicated Business Analysis & Product Specification showcase modeling the core mechanics of the **Akkuro Mortgage Platform** — the Dutch mortgage origination and advisory software ecosystem built by [Topicus](https://topicus.nl) (part of TSS / Constellation Software).

> [!NOTE]
> **Independent Domain Research & Demonstration Showcase:** This repository is an independent domain research project developed for technical demonstration and portfolio evaluation. All workflow models, calculation engines, and business rules are reconstructed exclusively from publicly accessible Dutch mortgage regulations (AFM, Nibud, NHG, Wft) and open market research. No proprietary or confidential information is used.

---

## 🎯 Purpose & Role Alignment

This repository demonstrates domain mastery, requirements engineering, and technical specification capabilities for the **Senior Business Analyst (Akkuro Mortgage Platform)** role at Topicus Vietnam.

Acting as a **technical bridge (Proxy BA)** between Dutch product owners, mortgage specialists, and Vietnam-based engineering & QA teams, this repository demonstrates:
1. **Statutory Domain Mastery:** Dutch mortgage regulations (AFM consumer protection, Nibud borrowing capacity tables, NHG guarantee caps, statutory LTV limits, and Annuity vs. Linear amortization).
2. **Process Architecture:** End-to-end 5-stage Loan Origination System (LOS) workflow modeling using BPMN 2.0 swimlanes, 9 finite state transitions (incorporating statutory exception states: `REJECTED`, `EXPIRED`, `CANCELLED`), and Four-Eyes segregation of duties.
3. **Agile Living Documentation:** High-traceability requirements engineering connecting the 5 Stages to 5 Epics with 5 Flagship User Stories (BDD Given-When-Then), and explicit rule IDs (`Applies BR-xxx`) mapped 1-to-1 to DMN Decision Tables.
4. **Data Architecture:** Conceptual Domain Model (ERD) and standardized Data Dictionary for 6 core entities bridging statutory Dutch terminology to technical data models.
5. **Interactive Prototype & Web Portal:** Zero-dependency browser portal integrating both the full FSD reader and a live compliance simulator.

---

## 🏛️ Repository Architecture (Living Documentation Model)

```text
akkuro-mortgage-spec-engine/
├── Akkuro_Mortgage_FSD_Complete.md        # Master Consolidated FSD (All 5 Chapters with Full Table of Contents)
├── docs/                                  # Modular Business Analysis Specifications
│   ├── 01_business_domain.md              # Market Context, AFM/Nibud/NHG Framework, Domain Glossary
│   ├── 02_process_architecture.md         # BPMN 2.0 Swimlanes, 9-State Transition Matrix & RBAC Four-Eyes
│   ├── 03_functional_requirements.md      # 5 Epics & 5 Flagship User Stories with BDD & Direct Rule Traceability
│   ├── 04_data_architecture.md            # Conceptual ERD & Standardized Data Dictionary (6 Entities)
│   └── 05_business_rules_matrix.md        # DMN Decision Tables (LTV, NHG, Nibud, Dossier Gates, Amortization)
├── demo/
│   └── index.html                         # Dual-Tab Interactive Web Portal (FSD Reader + Compliance Simulator)
└── README.md                              # Specification Overview & Alignment Guide
```

---

## 🇳🇱 5-Stage Loan Origination Lifecycle & Epic Traceability

| Stage | Process Phase ([02_process.md](file:///Users/hieu/Projects/akkuro-mortgage-spec-engine/docs/02_process_architecture.md)) | Functional Epic ([03_functional.md](file:///Users/hieu/Projects/akkuro-mortgage-spec-engine/docs/03_functional_requirements.md)) | Key Business Rules ([05_business_rules.md](file:///Users/hieu/Projects/akkuro-mortgage-spec-engine/docs/05_business_rules_matrix.md)) |
|:---:|---|---|---|
| **Stage 1** | Application Intake & Scenario Simulation (`DRAFT` $\rightarrow$ `SUBMITTED`) | **EPIC-01: Scenario Modeling & Intake**<br>*(US-01: Statutory Borrowing Capacity & Debt-Burden Gating)* | `BR-NIBUD-01..04`, `BR-CALC-01..02` |
| **Stage 2** | Dossier Intake & Document Verification (`SUBMITTED` $\rightarrow$ `DOCUMENT_VERIFICATION`) | **EPIC-02: Dossier Intake & Verification**<br>*(US-02: Mandatory 4-Document Intake & Income Verification Gate)* | `BR-DOC-01..02`, `BR-STATE-02` |
| **Stage 3** | Credit Assessment & Underwriting Gating (`DOCUMENT_VERIFICATION` $\rightarrow$ `CREDIT_ASSESSMENT` / `REJECTED`) | **EPIC-03: Credit Assessment & Compliance**<br>*(US-03: Statutory LTV & NHG Compliance Decision Gate)* | `BR-LTV-01..04`, `BR-NHG-01..04`, `BR-STATE-03` |
| **Stage 4** | Binding Offer & Statutory Reflection (`CREDIT_ASSESSMENT` $\rightarrow$ `BINDING_OFFER_ISSUED` / `EXPIRED`) | **EPIC-04: Binding Offer & Reflection**<br>*(US-04: Immutable Binding Offer & 14-Day Reflection Window)* | `BR-OFFER-01..02`, `BR-STATE-04` |
| **Stage 5** | Notary Deed Execution & Fund Settlement (`ACCEPTED_BY_BORROWER` $\rightarrow$ `NOTARY_PASSED` / `CANCELLED`) | **EPIC-05: Notary Closing & Settlement**<br>*(US-05: Notary Closing Package & Escrow Settlement)* | `BR-NOTARY-01..02`, `BR-STATE-05` |

---

## 💻 Interactive Portal & Simulator

To preview the full FSD specification and test calculation rules directly in your browser:
```bash
open demo/index.html
```
- **Tab 1 (📋 Living FSD Specification):** Full markdown specification formatted with navigation sidebar, BDD scenario cards, and rule badge links.
- **Tab 2 (🧮 Interactive Rule Simulator):** Live underwriting simulator evaluating LTV caps, NHG eligibility, guarantee fees, and 360-month Annuity vs. Linear amortization cash flows.
