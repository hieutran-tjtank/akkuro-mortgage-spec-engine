# 05. Business Rules & Financial Calculation Matrix

**System:** Akkuro Mortgage Platform (Loan Origination System)  
**Scope:** Statutory Calculation Rules, Decision Tables & Amortization Formulas  
**Standard:** DMN-Style Decision Matrices with Standardized Rule IDs  
**Author:** Senior Business Analyst  

---

## 1. Statutory Loan-to-Value (LTV) Rules

$$\text{LTV} = \left( \frac{\text{Total Requested Principal}}{\text{Appraised Market Value}} \right) \times 100\%$$

| Rule ID | EBV Included? | Max Statutory LTV Cap | Condition / Verification Requirement | Action / System Response |
|---|:---:|:---:|---|---|
| `BR-LTV-01` | No | **100.0%** | Requested loan $\le$ Appraised Market Value | Status = `COMPLIANT`. Normal underwriting proceeds. |
| `BR-LTV-02` | No | **100.0%** | Requested loan $>$ Appraised Market Value | Status = `REJECTED`. Hard stop: LTV violation flag logged. |
| `BR-LTV-03` | Yes | **106.0%** | EBV budget certified $\le$ 6% of Appraised Value | Status = `COMPLIANT`. Excess amount allocated to Construction Deposit (*Bouwdepot*). |
| `BR-LTV-04` | Yes | **106.0%** | Total loan $> 106.0\%$ of Appraised Value | Status = `REJECTED`. Hard stop: EBV ceiling exceeded. |

---

## 2. National Mortgage Guarantee (NHG) Decision Matrix

*Statutory Year: 2024 / 2025*

| Rule ID | Property Value / Purchase Price | Energy Saving Measures? | NHG Eligibility | Mandatory Fee (*Borgtochtprovisie*) | Interest Rate Impact |
|---|---|:---:|:---:|:---:|:---:|
| `BR-NHG-01` | $\le$ €435,000 | No | **Eligible** | $0.6\%$ of Principal | Automatic $-0.35\%$ rate discount |
| `BR-NHG-02` | $>$ €435,000 | No | **Ineligible** | Not applicable | Standard market rate applies |
| `BR-NHG-03` | $\le$ €461,100 | Yes (certified EBV) | **Eligible** | $0.6\%$ of Principal | Automatic $-0.35\%$ rate discount |
| `BR-NHG-04` | $>$ €461,100 | Yes | **Ineligible** | Not applicable | Standard market rate applies |

---

## 3. Nibud Borrowing Capacity & Debt Deduction Rules

| Rule ID | Debt / Obligation Type | Dutch Registry | Statutory Monthly Burden Ratio | Impact on Borrowing Capacity |
|---|---|---|---|---|
| `BR-NIBUD-01` | Base Borrowing Capacity | Nibud Tables | Maximum debt-to-income factor (typically 4.25x – 4.75x gross income depending on rate) | Establishes absolute ceiling on mortgage principal before debt adjustments. |
| `BR-NIBUD-02` | Revolving Consumer Credit | BKR | **2.0%** of total credit facility per month | Directly deducts 2% of credit limit from monthly disposable financing capacity. |
| `BR-NIBUD-03` | Student Loan (Pre-2015 / SF15) | DUO | **0.75%** of initial loan principal per month | High burden reduction due to 15-year repayment horizon. |
| `BR-NIBUD-04` | Student Loan (Post-2015 / SF35) | DUO | **0.35%** of initial loan principal per month | Moderate burden reduction due to 35-year repayment horizon. |

---

## 4. Dossier Verification & Underwriting Rules

| Rule ID | Rule Name | Gating Logic | System Action |
|---|---|---|---|
| `BR-DOC-01` | Mandatory 4-Document Checklist | All 4 documents (Employer Statement, UWV Record, NWWI Appraisal, Purchase Deed) must be marked `VERIFIED`. | Blocks transition from `DOCUMENT_VERIFICATION` to `CREDIT_ASSESSMENT`. |
| `BR-DOC-02` | Income Discrepancy Tolerance | Self-reported income variance against UWV insurance record must be $\le 5.0\%$. | If variance $> 5.0\%$, flags `CRITICAL_DISCREPANCY` and requires underwriter sign-off. |
| `BR-STATE-01` | Advisor Intake Gate | All required borrower and property fields must pass schema validation. | Allows transition `DRAFT` $\rightarrow$ `SUBMITTED`. |
| `BR-STATE-02` | Underwriting Entry Gate | Dossier status must be `VERIFIED`. | Allows transition `SUBMITTED` $\rightarrow$ `CREDIT_ASSESSMENT`. |
| `BR-STATE-03` | Credit Approval Gate | LTV compliant, loan $\le$ capacity, and underwriter four-eyes approval logged. | Allows transition `CREDIT_ASSESSMENT` $\rightarrow$ `BINDING_OFFER_ISSUED`. |
| `BR-STATE-04` | Offer Acceptance Gate | Borrower digital signature recorded within 14 calendar days. | Allows transition `BINDING_OFFER_ISSUED` $\rightarrow$ `ACCEPTED_BY_BORROWER`. |
| `BR-STATE-05` | Notary Deed Execution Gate | Notary deed executed and escrow fund settlement matched. | Allows transition `ACCEPTED_BY_BORROWER` $\rightarrow$ `NOTARY_PASSED`. |

---

## 5. Binding Offer & Notary Settlement Rules

| Rule ID | Rule Name | Specification | Legal & Audit Impact |
|---|---|---|---|
| `BR-OFFER-01` | Offer Immutability | Once issued, interest rates, monthly schedules, and loan tranches cannot be modified. | SHA-256 payload hash stored in audit ledger; any edit requires formal re-issuance. |
| `BR-OFFER-02` | Statutory Reflection Window | 14 calendar days (*Bedenktijd*) granted to borrower starting day after issuance. | Offer expires automatically on Day 15 if unexecuted; rate lock is revoked. |
| `BR-NOTARY-01` | Closing Instructions Transmission | Structured *Passeeropdracht* must be delivered to notary $\ge 10$ business days prior to closing date (*Passeerdatum*). | Ensures notary has adequate statutory window to draft mortgage deed (*Hypotheekakte*). |
| `BR-NOTARY-02` | Escrow Fund Settlement | Lender releases escrow mortgage funds only after receiving validated settlement statement (*Afleveringsnota*). | Eliminates escrow reconciliation deficit risks prior to notary signing. |

---

## 6. Repayment Amortization Formulas

### `BR-CALC-01`: Annuity Mortgage (*Annuïteitenhypotheek*)
Fixed monthly gross payment $M$ for $n = 360$ months at monthly rate $r = \frac{\text{Rate}_{\text{annual}}}{12}$:

$$M = P \cdot \frac{r(1+r)^n}{(1+r)^n - 1}$$

*For each month $k \in [1, n]$:*
1. **Interest Amount:** $I_k = \text{Balance}_{k-1} \cdot r$
2. **Principal Repayment:** $C_k = M - I_k$
3. **Closing Balance:** $\text{Balance}_k = \text{Balance}_{k-1} - C_k$

*Characteristics:* Gross installment is constant throughout 360 months. Interest component decreases monotonically as principal balance amortizes.

---

### `BR-CALC-02`: Linear Mortgage (*Lineaire hypotheek*)
Fixed monthly principal repayment $C$:

$$C = \frac{P}{n} = \frac{P}{360}$$

*For each month $k \in [1, n]$:*
1. **Principal Repayment:** $C_k = C$ (strictly constant)
2. **Interest Amount:** $I_k = \text{Balance}_{k-1} \cdot r$
3. **Total Gross Payment:** $M_k = C + I_k$
4. **Closing Balance:** $\text{Balance}_k = \text{Balance}_{k-1} - C$

*Characteristics:* Gross installment is highest in Month 1 and decreases every month. Minimizes lifetime total interest cost for the borrower.
