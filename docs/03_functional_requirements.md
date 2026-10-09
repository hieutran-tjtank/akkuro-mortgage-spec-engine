# 03. Functional Requirements & User Stories

**System:** Akkuro Mortgage Platform (Loan Origination System - LOS)  
**Standard:** Agile Epics, User Stories & Given-When-Then Acceptance Criteria  
**Traceability:** 1-to-1 Mapping across the 5 Process Stages ([02_process_architecture.md](file:///Users/hieu/Projects/akkuro-mortgage-spec-engine/docs/02_process_architecture.md)) and Business Rules ([05_business_rules_matrix.md](file:///Users/hieu/Projects/akkuro-mortgage-spec-engine/docs/05_business_rules_matrix.md))  
**Author:** Senior Business Analyst  

---

## 1. Requirements Breakdown Structure (5 Epics / 5 Flagship Stories)

```
EPIC-01: Loan Scenario Modeling & Application Intake (Stage 1)
└── US-01: Statutory Borrowing Capacity & Debt-Burden Gating [Applies BR-NIBUD-01..04, BR-CALC-01..02]

EPIC-02: Dossier Intake & Document Verification (Stage 2)
└── US-02: Mandatory 4-Document Intake & Income Verification Gate [Applies BR-DOC-01, BR-DOC-02, BR-STATE-02]

EPIC-03: Credit Assessment & Compliance Gating (Stage 3)
└── US-03: Statutory LTV & NHG Compliance Decision Gate [Applies BR-LTV-01..04, BR-NHG-01..04, BR-STATE-03]

EPIC-04: Binding Offer Issuance & Statutory Reflection (Stage 4)
└── US-04: Immutable Binding Offer Issuance & 14-Day Reflection Window [Applies BR-OFFER-01, BR-OFFER-02, BR-STATE-04]

EPIC-05: Notary Closing Instructions & Fund Settlement (Stage 5)
└── US-05: Notary Closing Package (Passeeropdracht) & Escrow Settlement [Applies BR-NOTARY-01, BR-NOTARY-02, BR-STATE-05]
```

---

## 2. Epic 01: Loan Scenario Modeling & Application Intake (Stage 1)

### US-01: Statutory Borrowing Capacity & Debt-Burden Gating
* **As a:** Mortgage Advisor (*Hypotheekadviseur*)  
* **I want to:** Input applicant gross incomes, interest rate fixing period, and existing debt obligations (DUO student debt, BKR credit lines)  
* **So that:** The system computes the statutory maximum borrowing capacity under AFM/Nibud regulations and models compliant annuity or linear repayment schedules.  
* **Applies Business Rules:** `BR-NIBUD-01`, `BR-NIBUD-02`, `BR-NIBUD-03`, `BR-NIBUD-04`, `BR-CALC-01`, `BR-CALC-02`

```gherkin
Scenario: Compliant capacity calculation with debt deduction
  Given an applicant with gross annual income of €65,000
  And an outstanding DUO student loan with initial balance of €20,000 (SF35 modern system)
  And an active BKR revolving credit facility limit of €2,000
  And a mortgage interest rate of 4.0% with a 10-year fixed term
  When the advisor calculates statutory borrowing capacity
  Then the DUO debt deducts €0.35 per €100 from monthly borrowing capacity (€70.00/month) per BR-NIBUD-02
  And the BKR facility deducts 2.0% of total credit limit from monthly budget (€40.00/month) per BR-NIBUD-03
  And the system outputs the net maximum permissible mortgage amount compliant with AFM anti-overcrediting rules
```

---

## 3. Epic 02: Dossier Intake & Document Verification (Stage 2)

### US-02: Mandatory 4-Document Intake & Income Verification Gate
* **As an:** Underwriting Intake Engine / Verification Specialist  
* **I want to:** Validate that all 4 mandatory statutory documents (Employer Statement, UWV Record, NWWI Appraisal, Purchase Agreement) are uploaded and verified without critical discrepancies  
* **So that:** Incomplete or unverified dossiers are strictly blocked from consuming lender underwriting capacity.  
* **Applies Business Rules:** `BR-DOC-01`, `BR-DOC-02`, `BR-STATE-02`

```gherkin
Scenario: Blocking incomplete dossier from advancing to underwriting
  Given a mortgage application in "DOCUMENT_VERIFICATION" status
  And the Employer Statement, UWV Record, and Purchase Agreement are marked "VERIFIED"
  But the NWWI Appraisal Report is missing or in "PENDING_UPLOAD" status
  When the advisor attempts to submit the dossier to Credit Assessment
  Then the system blocks the transition with error "DOSSIER_INCOMPLETE: Appraisal Report missing" per BR-DOC-01
  And the application state remains locked in "DOCUMENT_VERIFICATION"

Scenario: Income discrepancy reconciliation
  Given a verified UWV Insurance record showing 12-month earnings of €68,000 against self-reported salary of €75,000
  When the income variance exceeds 5.0%
  Then the system flags "CRITICAL_DISCREPANCY" per BR-DOC-02
  And requires an Employer Explanation Letter before verification sign-off is permitted
```

---

## 4. Epic 03: Credit Assessment & Compliance Gating (Stage 3)

### US-03: Statutory LTV & NHG Compliance Decision Gate
* **As a:** Lender Underwriter (*Acceptant*)  
* **I want to:** Automatically validate requested loan amounts against statutory LTV ceilings (100% standard vs. 106% with Energy Budget) and NHG guarantee eligibility criteria  
* **So that:** I can issue a compliant approval or formal statutory rejection with full regulatory auditability.  
* **Applies Business Rules:** `BR-LTV-01`, `BR-LTV-02`, `BR-LTV-03`, `BR-LTV-04`, `BR-NHG-01..04`, `BR-STATE-03`

```gherkin
Scenario: Compliant sustainable purchase with 106% LTV
  Given a property with appraised market value of €400,000
  And certified energy-saving measures (EBV) of €20,000
  When the applicant requests a total loan amount of €420,000 (LTV = 105.0%)
  Then the underwriting engine marks LTV as compliant (<= 106.0%) per BR-LTV-03
  And automatically allocates €20,000 into a designated construction deposit (Bouwdepot)

Scenario: Underwriter issues formal rejection on non-compliant loan
  Given a requested loan amount of €450,000 exceeding statutory Nibud borrowing capacity
  When the underwriter reviews the audit scorecard and triggers "Reject Application"
  Then the application transitions to terminal state "REJECTED"
  And the system logs the statutory violation reason in the immutable audit ledger
```

---

## 5. Epic 04: Binding Offer Issuance & Statutory Reflection (Stage 4)

### US-04: Immutable Binding Offer Issuance & 14-Day Reflection Window
* **As a:** Compliance Officer / Borrower (*Klant*)  
* **I want to:** Generate an immutable binding mortgage contract (*Definitieve Offerte*) and enforce the statutory 14-calendar-day reflection period (*Bedenktijd*)  
* **So that:** The borrower can execute a legally protected digital acceptance or allow the offer to expire without penalty.  
* **Applies Business Rules:** `BR-OFFER-01`, `BR-OFFER-02`, `BR-STATE-04`

```gherkin
Scenario: Borrower accepts binding offer within statutory window
  Given a binding offer issued on Day 1 with a 14-calendar-day reflection timer per BR-OFFER-02
  When the borrower electronically signs the acceptance on Day 7
  Then the application status transitions to "ACCEPTED_BY_BORROWER" per BR-STATE-04
  And the system initiates the civil-law notary closing package workflow

Scenario: Statutory reflection window expires without signature
  Given an active binding offer with Day 14 reflection window elapsed
  When Day 15 arrives without borrower digital signature
  Then the system automatically transitions the offer to "EXPIRED"
  And locks the file, requiring a new underwriting submission if the borrower wishes to proceed
```

---

## 6. Epic 05: Notary Closing Instructions & Fund Settlement (Stage 5)

### US-05: Notary Closing Package (Passeeropdracht) & Escrow Settlement
* **As a:** Operations Specialist / Civil-Law Notary (*Notaris*)  
* **I want to:** Transmit structured closing instructions (*Passeeropdracht*) to the notary and reconcile escrow settlement statements (*Afleveringsnota*) upon deed execution  
* **So that:** Mortgage deed execution and bank fund disbursement are synchronized seamlessly without closing escrow deficits.  
* **Applies Business Rules:** `BR-NOTARY-01`, `BR-NOTARY-02`, `BR-STATE-05`

```gherkin
Scenario: Automated notary instruction dispatch and escrow release
  Given an application in "ACCEPTED_BY_BORROWER" status
  When the calendar reaches 10 business days prior to scheduled closing date (Passeerdatum)
  Then the engine transmits the structured Passeeropdracht payload to the notary office per BR-NOTARY-01
  And upon notary confirmation of deed execution (Hypotheekakte), escrow funds are disbursed
  And the application state transitions to "NOTARY_PASSED" (Lifecycle Complete) per BR-STATE-05
```
