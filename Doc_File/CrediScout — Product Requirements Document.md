# **CrediScout — Product Requirements Document**

## **1\. Product Overview**

**Product:** CrediScout  
**Category:** Credit Analysis & Lending Decision Support  
**Primary User:** Credit Analysts  
**Initial Loan Products:**

* **Small Business Loans (SBL)** — Tiered RB rates by amount & client status  
* **SME Loans** — Tiered RB rates by amount & client status (same table as SBL)  
* **Agricultural Loans (AgroLoans)** — 5.5% RB flat across all amounts  
* **Clean Energy Loans** — 3.0% Flat Rate across all amounts  
* **Housing / Education Loans** — 3.0% Flat Rate across all amounts  
* **Asset Loans (AL)** — Rate per lender policy (configurable by Admin)

### **Product Vision**

CrediScout helps credit analysts transform borrower information and supporting documents into a **structured, transparent and evidence-based credit assessment**, culminating in an explicit lending recommendation.

The product is designed to **assist—not replace—the credit analyst**.

CrediScout should make it easier to answer:

> **Who is the borrower? What are they borrowing? Can they afford it? What risks exist? What amount can reasonably be supported? And what should the lender do?**

---

# **2\. User Roles**

CrediScout has **two user roles**:

---

### **2.1 Credit Analyst**

The primary user who performs credit assessments and produces lending recommendations.

A credit analyst uses CrediScout to perform deeper financial and credit/risk analysis and recommend an appropriate lending decision.

The analyst should be able to:

* Understand the borrower's profile.  
* Understand the purpose and structure of the requested facility.  
* Collect and validate relevant information.  
* Analyze financial capacity.  
* Analyze credit and repayment risk.  
* Assess business/borrower characteristics.  
* Evaluate collateral or security where applicable.  
* Consider loan-product-specific risks.  
* Identify red flags and mitigating factors.  
* Determine an appropriate loan amount, tenor, **and applicable lending rate**.  
* Produce an explicit credit recommendation.

---

### **2.2 Administrator (Admin)**

Administrators configure and govern the product. This role includes **product managers, designers, compliance officers, and system administrators**.

Admins can:

* Manage lending rate tables (SBL/SME tiered rates, product-specific flat rates)  
* Configure loan product definitions and risk parameters  
* Define credit policy thresholds (DSCR minimums, LTV caps, DTI limits)  
* Manage user accounts and analyst permissions  
* Review audit logs of assessments and overrides  
* Configure notification rules and escalation paths  
* Maintain reference data (industry codes, collateral types, document types)  
* View portfolio-level analytics and assessment quality metrics

> **Note:** Designers and product managers operate as Admins with appropriate permission scopes — they do not perform credit assessments.

---

# **3\. Problem**

Credit analysis can involve information scattered across application forms, bank statements, financial statements, credit records, business information, collateral documents and other supporting documents.

Analysts may therefore spend considerable time:

* Gathering and organizing information.  
* Performing repetitive calculations.  
* Comparing borrower capacity with requested exposure.  
* Identifying inconsistencies.  
* Assessing multiple risk factors.  
* Preparing credit recommendations.

There is also a risk that important indicators are overlooked or that assessments become inconsistent between applications.

### **CrediScout's solution**

CrediScout provides a **structured credit-analysis workflow** that brings relevant information, metrics, risk indicators and loan-specific considerations together in one assessment.

It should make the reasoning behind a recommendation visible to the analyst and relevant stakeholders.

---

# **4\. Product Goal**

CrediScout should enable a credit analyst to move from:

**Borrower information → Financial analysis → Risk analysis → Loan suitability → Recommended facility → Explicit decision**

The final output should not simply be a score.

It should clearly state a recommendation such as:

> **RECOMMENDATION: APPROVE**  
> Recommended Amount: ₦3,000,000  
> Recommended Tenor: 12 months  
> **Applicable Lending Rate: 5.00% RB per month** (Returning Client, SBL/SME tier ≤ ₦5m)  
> Key Reasons: Adequate repayment capacity, acceptable existing debt exposure and satisfactory business cash flow.  
> Key Conditions: Subject to satisfactory verification of specified documents.

Other possible outcomes include:

* **Approve**  
* **Approve at Reduced Amount**  
* **Decline**  
* **Refer for Further Review**

---

# **5\. Product Principles**

### **5.1 Human-in-the-loop**

CrediScout provides analysis and recommendations, but the credit analyst remains responsible for reviewing and validating important information.

### **5.2 Explainability**

Important metrics should not appear as unexplained numbers.

For example:

**Debt Service Coverage Ratio (DSCR): 1.45×**

> Measures whether available cash flow is sufficient to meet debt obligations. A ratio above 1 indicates that assessed cash flow exceeds the relevant debt-service requirement.

The exact interpretation or threshold may vary according to the lender's credit policy.

### **5.3 Evidence-based recommendations**

Every major recommendation should be supported by relevant financial, credit, borrower and loan information.

### **5.4 Product-specific analysis**

CrediScout should use a common credit-analysis framework while adapting relevant questions, indicators and risk considerations to the selected loan product.

### **5.5 Professional judgment**

The system should allow the analyst to identify circumstances that require human judgment rather than forcing every application into a rigid automated decision.

---

# **6\. Core User Journey**

## **Stage 1 — Create Credit Assessment**

The analyst starts a new borrower assessment.

The analyst identifies:

* Borrower  
* Borrower type  
* Loan product  
* Requested amount  
* Proposed tenor  
* Loan purpose  
* Repayment source  
* Existing exposure  
* **Client status: New / Returning** (determines applicable SBL/SME rate tier)  
* Other relevant application information

---

## **Stage 2 — Borrower & Loan Profile**

CrediScout establishes the context of the application before detailed analysis begins.

The profile should capture relevant information about:

### **Borrower**

* Individual/business identity  
* Business type  
* Business age  
* Industry/sector  
* Location  
* Management/ownership information  
* Relevant experience

### **Loan**

* Product type  
* Requested amount  
* Proposed tenor  
* Loan purpose  
* Repayment frequency where relevant  
* Proposed repayment source  
* Existing relationship/exposure  
* **Client status: New / Returning** (verified against borrower history — drives SBL/SME rate tier)

The information displayed should adapt to the selected loan product.

---

# **7\. Stage 3 — Information & Document Collection**

The analyst should be able to provide information through:

### **Manual entry**

For information obtained during assessment or field verification.

### **Document upload**

Potential documents may include:

* Bank statements  
* Financial statements  
* Payslips  
* Business records  
* Existing loan schedules  
* Tax/business documents  
* Property documents  
* Asset documents  
* Agricultural records  
* Valuation reports  
* Other supporting documents

### **Assisted information extraction**

Where appropriate, CrediScout can extract relevant information from submitted documents for analyst verification.

The analyst should be able to **review, correct and confirm extracted information**.

Important figures should not automatically become accepted facts simply because they were extracted from a document.

---

# **8\. Stage 4 — Financial Capacity Analysis**

CrediScout should assess whether the borrower has sufficient financial capacity to support the proposed facility.

Relevant metrics may include:

### **Revenue / Income**

Measures the borrower's primary income-generating capacity.

### **Operating Expenses**

Shows the recurring cost burden affecting available cash flow.

### **Net Income / Surplus**

Indicates the amount remaining after relevant expenses.

### **Cash Flow**

Shows the actual or estimated movement of funds available for repayment.

### **Debt-to-Income Ratio (DTI)**

Measures existing and/or proposed debt obligations relative to income.

### **Debt Service Coverage Ratio (DSCR)**

Measures the borrower's ability to cover debt-service obligations from available cash flow.

### **Loan-to-Income / Loan-to-Revenue**

Provides context on the size of the requested facility relative to the borrower's earning or business capacity.

### **Existing Debt Exposure**

Shows the borrower's current obligations before considering the proposed facility.

### **Proposed Debt-Service Burden**

Shows how the proposed facility would affect the borrower's repayment obligations.

CrediScout should clearly distinguish between:

* Existing obligations  
* Proposed obligation  
* Total obligation after the proposed loan

---

# **9\. Stage 5 — Credit & Risk Analysis**

CrediScout should identify factors that could affect repayment risk.

Potential areas include:

### **Repayment History**

Previous performance on loans or credit facilities.

### **Delinquency / Default Indicators**

Evidence of late repayment, arrears or previous default.

### **Existing Credit Exposure**

Total obligations across relevant facilities.

### **Credit Utilization**

How heavily existing credit facilities are being utilized where relevant.

### **Multiple Borrowing**

Potential signs of excessive borrowing or obligations across lenders.

### **Guarantor Obligations**

Existing obligations where the borrower has guaranteed another person's facility.

### **Credit Red Flags**

Potential warning indicators requiring analyst attention.

The product should distinguish between:

**Risk identified** → **Why it matters** → **Potential mitigating factor**

rather than simply displaying a red warning.

---

# **10\. Stage 6 — Borrower / Business Analysis**

CrediScout should assess qualitative factors that financial ratios alone cannot capture.

Potential areas include:

* Business operating history  
* Industry/sector  
* Management experience  
* Business stability  
* Customer concentration  
* Supplier concentration  
* Business model  
* Revenue stability  
* Seasonality  
* Purpose of borrowing  
* Source of repayment  
* Management quality  
* Relevant business risks

For individual borrowers, relevant personal financial and employment information may replace business-specific indicators.

---

# **11\. Stage 7 — Collateral & Security Analysis**

Where collateral/security is relevant, CrediScout should assess:

* Type of security  
* Ownership  
* Estimated value  
* Verified value where applicable  
* Marketability  
* Existing encumbrances  
* Loan-to-value ratio  
* Security coverage  
* Documentation status

### **Loan-to-Value (LTV)**

Shows the relationship between the proposed exposure and the value of the relevant asset/security.

CrediScout should explain that a higher LTV generally means greater exposure relative to the underlying asset value, subject to the lender's policy and the nature of the asset.

Collateral should **not automatically override poor repayment capacity**.

---

# **12\. Stage 8 — Loan-Specific Analysis**

CrediScout should adapt the assessment according to the selected loan product.

## **Small Business Loans (SBL)**

Potential focus:

* Business cash flow  
* Revenue stability  
* Operating expenses  
* Existing obligations  
* Business history  
* Loan purpose  
* Repayment source  
* DSCR  
* Business risks  
* **Applicable rate tier (New/Returning, amount band)**

## **SME Loans**

Potential focus:

* Financial statements  
* Revenue and profitability  
* Working capital  
* Cash flow  
* Existing exposure  
* Business structure  
* Management  
* Industry risk  
* DSCR  
* Debt burden  
* **Applicable rate tier (New/Returning, amount band)**

## **Agricultural Loans (AgroLoans)**

Potential focus:

* Farm/agribusiness profile  
* Production cycle  
* Seasonality  
* Expected production  
* Expected revenue  
* Input costs  
* Existing obligations  
* Market conditions  
* Repayment source  
* Agricultural-specific risks  
* **Fixed rate: 5.5% RB (all amounts, all client types)**

## **Clean Energy Loans**

Potential focus:

* Project type (solar, battery, efficiency, etc.)  
* Expected savings / revenue generation  
* Technical feasibility  
* Installation/maintenance costs  
* Existing obligations  
* Repayment source (energy savings, feed-in tariff, etc.)  
* **Fixed rate: 3.0% Flat (all amounts, all client types)**

## **Housing / Education Loans**

Potential focus:

* Borrower income  
* Household affordability  
* Existing obligations  
* Proposed housing/education payment  
* Property value (for housing)  
* Loan-to-value (for housing)  
* Repayment capacity  
* Employment/income stability  
* **Fixed rate: 3.0% Flat (all amounts, all client types)**

## **Asset Loans (AL)**

Potential focus:

* Asset type  
* Asset value  
* Asset purpose  
* Borrower repayment capacity  
* Asset productivity/income contribution  
* Loan-to-value  
* Asset condition  
* Ownership/documentation  
* Security coverage  
* **Rate per lender policy (Admin-configurable)**

---

# **13\. Stage 9 — Risk & Red-Flag Summary**

Before producing the final recommendation, CrediScout should provide a concise risk summary.

### **Example**

**Key Strengths**

* Stable business revenue  
* Positive operating cash flow  
* Adequate debt-service capacity

**Key Risks**

* High existing debt exposure  
* Revenue concentration  
* Short repayment history

**Mitigating Factors**

* Strong collateral coverage  
* Long-standing business operation  
* Verified alternative repayment source

This gives stakeholders a quick understanding of **why the application is supportable or problematic**.

---

# **14\. Stage 10 — Recommended Loan Amount**

This is one of CrediScout's most important features.

The application should not simply answer:

> “Can this borrower receive the requested amount?”

It should also help answer:

> **“What amount can reasonably be supported based on the assessed information?”**

CrediScout should compare:

**Requested Amount vs. Assessed Capacity vs. Relevant Risk Factors**

The result may be:

* Requested amount supported  
* Lower amount supported  
* No supportable amount identified  
* Further review required

Example:

> Requested Amount: ₦5,000,000  
> Assessed Supportable Amount: ₦3,200,000  
> Recommendation: Approve ₦3,200,000

The analyst should be able to understand the major factors driving the difference.

---

# **15\. Stage 11 — Final Credit Recommendation**

The final recommendation should be **explicit**.

### **Recommended decision structure**

**Decision:** APPROVE / APPROVE AT REDUCED AMOUNT / DECLINE / REFER

**Recommended Amount:** ₦X

**Recommended Tenor:** X months

**Applicable Lending Rate:** X% [RB / Flat] per month

**Repayment Structure:** Relevant repayment frequency/structure (e.g., monthly, quarterly, bullet)

**Key Reasons:**  
Short explanation of the major factors supporting the decision.

**Key Risks:**  
Important identified risks.

**Mitigating Factors:**  
Factors that reduce or address identified risks.

**Conditions:**  
Requirements that should be satisfied before or during approval where applicable.

### **Example**

> **RECOMMENDATION: APPROVE AT REDUCED AMOUNT**

> Requested Amount: ₦5,000,000  
> Recommended Amount: ₦3,200,000  
> Recommended Tenor: 12 months  
> **Applicable Lending Rate: 5.00% RB per month** (Returning Client, SBL/SME tier ≤ ₦5m)

> **Basis:** Assessed repayment capacity supports a lower exposure than requested. Existing obligations and cash-flow variability limit the supportable facility.

> **Key Risks:** Existing debt burden and revenue variability.

> **Mitigating Factors:** Established operating history and positive cash flow.

> **Conditions:** Satisfactory verification of supporting financial information and required security documentation.

---

# **16\. Analyst Override & Professional Judgment**

CrediScout should allow the analyst to disagree with the system's suggested recommendation.

If the analyst changes the recommendation, the product should ask for a reason.

Example:

**CrediScout Recommendation:** Approve ₦3.2m

**Analyst Recommendation:** Approve ₦4m

**Reason:** Verified additional repayment source not reflected in the initial financial assessment.

This ensures that CrediScout supports professional judgment rather than pretending that every lending decision can be determined automatically.

---

# **17\. Lending Rate Determination**

CrediScout automatically determines the applicable lending rate based on **loan product**, **recommended amount**, **client status** (new vs. returning), and **rate type** (Reducing Balance vs. Flat Rate). The rate is displayed in the final recommendation and used for repayment schedule calculations.

## **17.1 SBL / SME Loans — Tiered Reducing Balance (RB) Rates**

Rates vary by loan amount band and client relationship status.

### **New Clients**

| Loan Amount Band | Rate (RB per month) |
|------------------|---------------------|
| ≤ ₦5,000,000 | 5.00% |
| > ₦5,000,000 – ₦9,999,999 | 4.65% |
| ≥ ₦10,000,000 – ₦19,999,999 | 4.60% |
| ≥ ₦20,000,000 – ₦29,999,999 | 4.50% |
| ≥ ₦30,000,000 – ₦49,999,999 | 4.25% |
| ≥ ₦50,000,000 | 4.00% |

### **Returning Clients**

| Loan Amount Band | Rate (RB per month) |
|------------------|---------------------|
| ≤ ₦5,000,000 | 5.00% |
| > ₦5,000,000 – ₦9,999,999 | 4.60% |
| ≥ ₦10,000,000 – ₦19,999,999 | 4.50% |
| ≥ ₦20,000,000 – ₦29,999,999 | 4.40% |
| ≥ ₦30,000,000 – ₦49,999,999 | 4.20% |
| ≥ ₦50,000,000 | 3.50% |

> **Rule:** The **recommended amount** (not the requested amount) determines the applicable band.  
> **Rule:** "Returning Client" = borrower with at least one fully repaid prior facility in good standing, or an existing performing facility with ≥ 6 months clean repayment history.

---

## **17.2 Fixed-Rate Products — Flat Rate (All Amounts, All Client Types)**

| Loan Product | Rate (Flat per month) | Applicability |
|--------------|----------------------|---------------|
| **Agricultural Loans (AgroLoans)** | 5.5% RB | All amounts, new & returning |
| **Clean Energy Loans** | 3.0% Flat | All amounts, new & returning |
| **Housing / Education Loans** | 3.0% Flat | All amounts, new & returning |

> **Note on Rate Types:**  
> - **RB (Reducing Balance):** Interest calculated on outstanding principal each period.  
> - **Flat Rate:** Interest calculated on original principal for the full tenor.  
> CrediScout must clearly label which method applies so the analyst and borrower understand the effective cost difference.

---

## **17.3 Rate Application Logic**

1. **Analyst selects loan product** → CrediScout loads the correct rate table.
2. **Analyst records client status** (New / Returning) — verified against borrower history.
3. **System calculates recommended amount** (Stage 10).
4. **System looks up rate** from the appropriate table using the recommended amount band.
5. **Rate is displayed** in the Final Recommendation (Stage 11) with rate type (RB/Flat).
6. **Analyst may override rate** only with documented justification and Admin approval (audit-logged).

---

## **17.4 Repayment Schedule Preview**

Once the rate is determined, CrediScout generates a **repayment schedule preview** showing:

* Monthly/quarterly installment amount
* Principal vs. interest breakdown per period
* Total interest payable over the tenor
* Effective annual rate (for transparency)

This preview is included in the **Credit Assessment Summary** (Section 21) and the **Executive Summary** (Section 18).

---

# **18\. Stakeholder View**

The final assessment should provide different levels of information depending on the stakeholder's needs.

### **Executive Summary**

A decision-maker should quickly see:

* Borrower  
* Loan product  
* Requested amount  
* Recommended amount  
* Recommended decision  
* Key risk level/indicators  
* Key reasons  
* Conditions  
* Analyst recommendation

### **Detailed Analysis**

A credit professional should be able to inspect:

* Financial metrics  
* Credit metrics  
* Borrower/business analysis  
* Collateral  
* Product-specific indicators  
* Supporting evidence  
* Risks  
* Mitigating factors  
* Analyst comments

This prevents stakeholders from having to interpret dozens of metrics before understanding the actual recommendation.

---

# **19\. Metrics & Explanations**

CrediScout should maintain a consistent pattern for important metrics:

**Metric → Value → Meaning → Interpretation**

Example:

**DSCR: 1.45×**

**What it measures:**  
Ability of assessed cash flow to cover debt-service obligations.

**What it means in this assessment:**  
The assessed cash flow is 1.45 times the relevant debt-service requirement.

**Interpretation:**  
The result should be considered against the lender's applicable credit policy and other risk factors.

This approach should be used throughout the application.

---

# **20\. Alerts & Exceptions**

CrediScout should highlight issues that require analyst attention.

Examples:

* Missing critical information  
* Inconsistent financial information  
* Significant increase in reported obligations  
* High debt burden  
* Weak repayment capacity  
* Poor repayment history  
* Excessive requested exposure  
* Unverified documents  
* Security/documentation concerns  
* Significant deviation from expected business performance

Alerts should explain **why the issue matters**, rather than merely displaying a warning.

---

# **21\. Credit Assessment Summary**

At the end of every assessment, CrediScout should generate a concise credit-analysis summary containing:

1. Borrower profile  
2. Loan request  
3. Financial position  
4. Repayment capacity  
5. Credit profile  
6. Business/borrower assessment  
7. Security/collateral assessment  
8. Product-specific assessment  
9. Key strengths  
10. Key risks  
11. Mitigating factors  
12. Recommended amount  
13. Recommended tenor  
14. **Applicable lending rate (rate % + RB/Flat + client status basis)**  
15. Repayment schedule preview (installment, total interest, effective annual rate)  
16. Final recommendation  
17. Conditions  
18. Analyst comments

---

# **22\. MVP Scope**

The first version should focus on the **core credit-analysis journey**, rather than attempting to build a complete lending-management platform.

### **MVP should include:**

* Borrower & Loan Profile  
* Six loan-product categories  
* Manual information entry  
* Document submission  
* Information verification  
* Financial analysis  
* Credit/risk analysis  
* Borrower/business analysis  
* Collateral/security assessment where applicable  
* Product-specific indicators  
* Key metrics with explanations  
* Risk/red-flag identification  
* Recommended loan amount  
* **Automated lending rate determination (tiered RB for SBL/SME, flat rates for Agro/Clean Energy/Housing-Education)**  
* Explicit lending recommendation (with rate, tenor, repayment schedule preview)  
* Analyst override (decision, amount, tenor, **rate — with justification**)  
* Reasons/conditions  
* Credit assessment summary  
* **Loan disbursement — handles and disburses approved loans to borrower bank accounts and confirms payment status via APIs/webhooks**

### **Defer initially**

Features such as:

* Collections management  
* Full loan servicing  
* Customer-facing loan applications  
* Portfolio management  
* Institution-wide core banking functions  
* Complex workflow administration

These can be considered after the core credit-analysis product has been validated.

---

# **23\. Success Criteria**

CrediScout should ultimately demonstrate that a credit analyst can:

**1\. Understand an application faster**  
The relevant borrower and loan information is organized in one place.

**2\. Perform a more structured assessment**  
Important financial, credit, business and security considerations are systematically evaluated.

**3\. Understand the numbers**  
Important metrics are accompanied by concise explanations.

**4\. Identify important risks**  
Material red flags are brought to the analyst's attention.

**5\. Determine a supportable exposure**  
The analyst can compare the requested amount with assessed repayment capacity and risk.

**6\. Produce an explicit recommendation**  
The final output clearly states whether to approve, approve at a reduced amount, decline, or refer for further review — **including the applicable lending rate, tenor, and repayment schedule**.

**7\. Apply correct pricing automatically**  
The system selects the correct rate from configured tables (SBL/SME tiered RB by amount band and client status; flat rates for Agro/Clean Energy/Housing-Education) based on the **recommended amount** and **verified client status**.

**8\. Explain the recommendation**  
A stakeholder can understand the principal evidence and reasoning behind the recommendation.

---

# **24\. Core Product Proposition**

> **CrediScout helps credit analysts turn borrower information into structured financial and risk analysis, identify key credit risks, determine a supportable loan amount, apply the correct lending rate, produce an explicit, evidence-based lending recommendation with a full repayment schedule — and disburse approved loans to borrower bank accounts, confirming payment status via APIs/webhooks.**

### **Product philosophy**

**Don't just calculate credit metrics.**

**Understand the borrower.**  
**Understand the loan.**  
**Understand the risk.**  
**Determine the supportable exposure.**  
**Price it correctly.**  
**Explain the decision.**  
**Disburse what you approve.**

