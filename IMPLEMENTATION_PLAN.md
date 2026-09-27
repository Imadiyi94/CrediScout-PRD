# CrediScout — Detailed Implementation Plan (Phased)

Source of truth: `Doc_File/CrediScout — Product Requirements Document.md` (24 sections, incl. §17 Lending Rate Determination).
This plan translates the PRD into buildable phases: design system → architecture → data → workflow stages → calculation engines → recommendation → summaries → hardening → launch.

---

## 0. What We Are Building (distilled from the PRD)

- **Users (2 roles only):** `ANALYST` (credit assessments) and `ADMIN` (configures everything; includes managers/designers/compliance with scoped permissions).
- **Loan products (6):**
  1. SBL — tiered RB by amount × client status
  2. SME — same tiered RB table as SBL
  3. AgroLoans — 5.5% RB, all amounts, all clients
  4. Clean Energy — 3.0% Flat, all amounts, all clients
  5. Housing / Education — 3.0% Flat, all amounts, all clients
  6. Asset Loans (AL) — Admin-configurable rate
- **11-stage workflow:** Create assessment → Borrower & Loan Profile → Info/Docs → Financial capacity → Credit/risk → Borrower/business → Collateral → Product-specific → Risk summary → Recommended amount → Final recommendation (amount + tenor + rate + schedule + reasons/risks/mitigations/conditions), plus override, stakeholder views, assessment summary.
- **Pricing rule:** the **recommended amount** (not requested) + verified **client status** + **product** determine the rate. Rate override requires justification + Admin approval, audit-logged.
- **MVP excludes:** disbursement, collections, servicing, customer-facing applications, portfolio management, core banking, complex workflow admin.

> ⚠️ **PRD inconsistency to resolve before build (decision required):** §22 MVP says "Five loan-product categories" but §1/§12 now list **six** (SBL, SME, Agro, Clean Energy, Housing/Education, Asset). **Recommendation:** build six; fix the "five" wording in the PRD.

---

## Phase 0 — Foundations & Architectural Decisions

### 0.1 Stack decision (recommended for a beginner team, single deployable)

| Layer | Choice | Why |
|---|---|---|
| App | **Next.js 15 (App Router) + TypeScript** monolith | One repo, one deploy; server components for analyst screens, route handlers for API |
| Styling/UI | **Tailwind CSS + shadcn/ui** | Accessible primitives, fast to theme, matches "structured workflow" UX |
| ORM/DB | **Prisma + PostgreSQL 16** | Strong relational model (borrowers→assessments→schedules); migrations in git |
| Auth | **Auth.js (NextAuth v5)** credentials + sessions | Only 2 roles; no external IdP needed for MVP |
| Validation | **Zod** (shared server/client schemas) | Single source of truth for forms + API |
| Money | **Integer kobo** in DB + `decimal.js` in engine | Avoids float errors on ₦ amounts and interest |
| File storage | **S3-compatible** (MinIO locally via Docker, S3/R2 in prod) | PRD requires document submission + verification |
| Calculation core | **Pure-TS package `packages/credit-engine/`** | Deterministic, unit-testable, auditable; no framework imports |
| PDF/export | Server-rendered summary → PDF (e.g. `@react-pdf/renderer` or headless print CSS) | Stakeholder/executive summary must be shareable |
| Infra (local) | **Docker Compose:** `web`, `postgres`, `minio` | Matches installed Docker CLI/Compose; reproducible dev env |
| CI/CD | **GitHub Actions:** lint → typecheck → unit tests → build → migrate check | Repo already on GitHub; branch protection on `master`/`main` |

### 0.2 Architecture shape (modular monolith)

```
/app                    # Next.js routes (analyst workflow, admin, summaries)
/components             # design-system + domain components
/lib                    # auth, db client, storage, pdf
/packages/credit-engine # DTI/DSCR/LTV, RB + flat schedules, rate resolver, supportable-amount solver
/prisma                 # schema + migrations + seeds (rate tables, thresholds, reference data)
/docker                 # Compose files, Dockerfiles
```

- **Module boundaries:** `borrowers`, `assessments`, `documents`, `financials`, `credit-risk`, `collateral`, `products`, `pricing`, `recommendations`, `audit`.
- **Rule:** UI never computes money logic inline — it calls `credit-engine` (same functions used by API + tests).
- **Audit:** append-only `AuditEvent` table; every create/update/override/rate-change writes an event with actor, before/after, reason.

### 0.3 Environments & branch strategy

- Branches: `master` (or rename to `main`) = deployable; feature branches `feat/<phase>-<slug>`; PRs with checklist.
- Envs: `dev` (Docker Compose), `staging`, `prod`. Secrets via env vars (never committed). `gh secret set` for CI.
- Definition of done per phase: migrated DB + seeded reference data + engine unit tests green + UI reachable + audit events verified.

### 0.4 Phase 0 tasks

1. Rename default branch if desired (`master` → `main`) and set GitHub branch protection.
2. Scaffold Next.js + TS + Tailwind + shadcn/ui + Prisma + Auth.js skeleton.
3. Add Docker Compose (`web`, `postgres:16`, `minio`), `.env.example`, health-check route.
4. Add GitHub Actions workflow (install → lint → typecheck → `vitest` → build).
5. Seed script stubs (rate tables in §17, roles, one demo analyst + admin).
6. **Acceptance:** `docker compose up` → app loads, login works, `prisma migrate dev` clean, CI green.

---

## Phase 1 — Design System & UX Shell

Goal: every PRD screen follows two repeatable patterns — **Metric → Value → Meaning → Interpretation** (§19) and **Risk → Why it matters → Mitigating factor** (§9).

### 1.1 Design tokens

- Colors: `background/surface/border`, `primary` (trust navy), `success/warning/danger` (decision + risk levels), `muted` text. Dark-mode-ready tokens from day one.
- Typography: display (recommendation headline), body, numeric tabular figures for ₦ amounts (`font-variant-numeric: tabular-nums`).
- Spacing/radius/shadows: 4pt scale; cards for each workflow stage; sticky summary rail on assessment pages.
- ₦ formatting utility: `formatNGN(kobo)` → `₦3,200,000`; never show raw kobo.

### 1.2 Component inventory (build once, reuse everywhere)

- Primitives: Button, Input, Select, Textarea, DatePicker, CurrencyInput (₦, kobo-safe), RadioGroup, Checkbox, Badge, Tabs, Accordion, Table, Pagination, Dialog/Sheet, Toast, Tooltip, FileUploader, EmptyState, Skeleton.
- Domain components:
  - `AssessmentStepper` (11 stages with statuses: not-started/in-progress/complete/needs-attention).
  - `MetricCard` (props: `title, value, meaning, interpretation, source`) — enforces §19 pattern.
  - `RiskRow` (props: `risk, whyItMatters, mitigation, severity`) — enforces §9 pattern.
  - `RateBadge` (e.g. `4.60% RB / mo · Returning · SBL ₦5–9.9m band`).
  - `RepaymentScheduleTable` (period, opening, payment, principal, interest, closing + totals + EAR).
  - `RecommendationPanel` (decision + amount + tenor + rate + reasons/risks/mitigations/conditions).
  - `OverrideDialog` (old vs new + mandatory reason field).
  - `AlertBanner` (issue + why-it-matters + link to fixing stage).
  - `ExecutiveSummaryCard` vs `DetailedAnalysisTabs` (§18 stakeholder views).

### 1.3 App shell & navigation

- Analyst: Dashboard (pipeline: Draft → In review → Recommended → Decided), Borrowers, Assessments, Documents queue, Tasks/alerts.
- Admin: Rate tables, Products, Policy thresholds, Users, Reference data, Audit log, Analytics (read-only MVP).
- Layout: left nav + top context bar (borrower · product · requested vs recommended) + right summary rail on assessment pages.
- **Acceptance:** Storybook (or `/design` preview route) shows all components with ₦ examples; stepper navigates 11 empty stages.

---

## Phase 2 — Data Model, Auth/RBAC, Admin Configuration

### 2.1 Prisma schema (core tables)

```
User(id, name, email, passwordHash, role: ANALYST|ADMIN, scopes[], active)
Borrower(id, type: INDIVIDUAL|BUSINESS, displayName, contacts, businessProfile{...}, clientStatus: NEW|RETURNING, clientStatusVerifiedAt, clientStatusVerifiedBy)
Assessment(id, borrowerId, analystId, product: SBL|SME|AGRO|CLEAN_ENERGY|HOUSING_EDU|ASSET, status, requestedAmountKobo, proposedTenorMonths, loanPurpose, repaymentSource, repaymentFrequency, existingExposureKobo, currentStage, createdAt, updatedAt)
FinancialSnapshot(assessmentId, revenueKobo, opexKobo, netIncomeKobo, cashFlowKobo, existingDebtServiceKobo, proposedDebtServiceKobo, dti, dscr, loanToIncome, verified: boolean, verifiedBy, notes)
CreditProfile(assessmentId, repaymentHistory, delinquencyFlags, totalExposureKobo, utilizationPct, multipleBorrowingFlags, guarantorObligationsKobo, redFlags[])
BusinessProfile(assessmentId, operatingHistoryMonths, industryCode, managementExperience, stabilityNotes, concentrationFlags, seasonality, purposeFit, repaymentSourceFit)  // personal-employment variant for individuals
Collateral(id, assessmentId, type, ownership, estimatedValueKobo, verifiedValueKobo, marketability, encumbrancesKobo, documentationStatus, ltv)
ProductAssessment(assessmentId, product, payloadJson, notes)   // per-§12 product-specific fields
RateTable(id, product, clientStatus, minAmountKobo, maxAmountKobo, ratePctMonthly, rateType: RB|FLAT, effectiveFrom, effectiveTo, active)
PolicyThreshold(id, key[DSCR_MIN, DTI_MAX, LTV_MAX_BY_PRODUCT...], value, product?)
Recommendation(id, assessmentId, decision: APPROVE|REDUCED|DECLINE|REFER, requestedAmountKobo, recommendedAmountKobo, recommendedTenorMonths, ratePctMonthly, rateType, rateBasis, installmentKobo, totalInterestKobo, effectiveAnnualRatePct, reasons, risks[], mitigations[], conditions[], analystNotes, decidedBy, decidedAt)
RecommendationOverride(id, recommendationId, field, oldValue, newValue, reason, approvedByAdmin?, createdBy)
Document(id, assessmentId, kind, fileKey, originalName, mimeType, sizeBytes, extractedJson?, verificationStatus: PENDING|VERIFIED|REJECTED, verifiedBy, verifiedAt)
Alert(id, assessmentId, code, severity, message, whyItMatters, stage, resolvedAt)
AuditEvent(id, actorId, assessmentId?, action, entityType, entityId, beforeJson, afterJson, reason?, createdAt)
```

### 2.2 Auth & permissions matrix

| Capability | Analyst | Admin |
|---|---|---|
| Create/edit assessments, upload docs, run analyses | ✅ | read-only |
| Override decision/amount/tenor | ✅ with reason | — |
| Override **rate** | propose only | ✅ approve |
| Edit rate tables, products, thresholds, users, reference data | ❌ | ✅ (scoped: designers/PMs get config-only scopes, no user mgmt) |
| View audit log / analytics | own assessments | ✅ all |

### 2.3 Admin screens (CRUD + versioning)

- Rate tables editor with **effective dating** (never hard-delete; expire + insert new row) and a **dry-run preview** ("rate for ₦X as New/Returning per product").
- Policy thresholds editor (DSCR min, DTI max, LTV caps per product).
- Reference data: industry codes, collateral types, document kinds, alert codes.
- **Seed data (from PRD §17):** SBL/SME 6-band × New/Returning tables; Agro 5.5% RB open-ended row; Clean Energy 3.0% Flat; Housing/Edu 3.0% Flat; Asset placeholder row flagged "configure before use".
- **Acceptance:** changing a rate creates a new effective row + audit event; old recommendations keep their historical rate (immutability).

---

## Phase 3 — Assessment Workflow Stages 1–3 (intake, profile, documents)

Maps to PRD §6–§7.

1. **Stage 1 — Create assessment:** borrower picker (or quick-create), borrower type, product, requested amount (₦), proposed tenor, purpose, repayment source/frequency, existing exposure, **client status New/Returning** (flag "unverified" until Stage 2 check).
2. **Stage 2 — Borrower & Loan Profile:** adaptive form per product (business fields for SBL/SME/Agro; employment/income for Housing/Edu individuals; project fields for Clean Energy; asset fields for AL). Client-status verification panel (prior facilities / 6-month clean history checklist → Verified toggle + audit).
3. **Stage 3 — Documents:** upload by kind (bank statements, financials, payslips, tax docs, property/asset docs, agro records, valuations); **no auto-accept** — each doc needs analyst Verify/Reject; verification checklist gates later stages ("3 of 5 critical docs verified").
4. Manual-entry forms for field-verification data; everyistem number shows source (manual vs doc-verified).
5. **Acceptance:** can create all 6 product types end-to-end through Stage 3; unverified client status blocks rate resolution with a clear alert.

---

## Phase 4 — Financial Capacity Engine (Stage 4, PRD §8)

### 4.1 `credit-engine` functions (pure, tested)

- `dti(existing+proposed service, income)`, `dscr(cashFlow, debtService)`, `loanToIncome()`, `exposureSplit(existing, proposed)`.
- `rbSchedule(principalKobo, monthlyRatePct, tenorMonths, frequency)` → equal-instalment amortisation, per-period split, totals.
- `flatSchedule(principalKobo, monthlyFlatPct, tenorMonths, frequency)` → interest on original principal.
- `effectiveAnnualRate(schedule)` for transparency.
- `supportableAmount({cashFlow, existingService, dscrMin, ratePct, rateType, tenor, roundTo: 50_000})` — binary-search max principal satisfying DSCR ≥ policy min. This powers "Requested vs Supportable".

### 4.2 UI

- Financial input form (revenue, opex, net, cash flow, existing service) with source tags.
- Results panel: each metric as `MetricCard` (value + meaning + interpretation vs policy threshold).
- Existing vs Proposed vs Total obligation split bar.
- **Acceptance:** ≥30 unit tests incl. RB schedule totalling to principal, flat > RB total for same nominal rate, solver respects DSCR floor; UI shows DSCR 1.45×-style explanations.

---

## Phase 5 — Credit & Risk Engine (Stage 5, PRD §9)

- Structured inputs: repayment history grade, delinquency/default flags, total exposure, utilization %, multiple-borrowing signals, guarantor obligations.
- Rule-based flag generator → each flag emits `{code, severity, whyItMatters, suggestedMitigation, stage}` rendered as `RiskRow` (never a bare red badge).
- Flags feed Stage 9 summary + recommendation reasons automatically.
- **Acceptance:** fixture borrowers produce expected flag sets; every flag has non-empty why/mitigation text.

---

## Phase 6 — Qualitative, Collateral & Product-Specific (Stages 6–8, PRD §10–§12)

1. **Stage 6:** scored questionnaire (operating history, industry, management, stability, concentration, seasonality, purpose/source fit) → qualitative score + narrative notes; individual-borrower variant (employment/income stability).
2. **Stage 7:** collateral register (multi-item) with estimated vs verified values, encumbrances, marketability, doc status; auto LTV per item + coverage ratio; hard rule — **collateral never overrides failed DSCR** (UI + engine enforce; attempting to approve with DSCR < min triggers blocking alert).
3. **Stage 8 per-product panels** (§12 + rate hint):
   - SBL/SME: cash-flow/debt-burden + rate-tier hint.
   - Agro: cycle/seasonality/expected yield + 5.5% RB note.
   - Clean Energy: project type/savings/feasibility + 3.0% Flat note.
   - Housing/Edu: affordability + LTV + 3.0% Flat note.
   - Asset: asset condition/ownership/productivity + Admin-configured rate (block if unconfigured).
4. **Acceptance:** one complete fixture per product passes through Stages 6–8 with product-specific fields persisted.

---

## Phase 7 — Recommendation, Pricing & Override (Stages 9–11 + §16–§17)

### 7.1 Stage 9 — Risk & Red-Flag Summary (auto-compiled)

- Strengths (from financials + history), Risks (from flags + thresholds), Mitigations (analyst-editable). One-screen "why supportable / why problematic".

### 7.2 Stage 10 — Recommended amount

- Compare Requested vs Supportable (solver output) vs Risk adjustment; outcomes: full / reduced / none / refer. Show "major factors driving the difference" list.

### 7.3 Pricing (§17.1–§17.3) — `resolveRate({product, recommendedAmountKobo, clientStatus, effectiveDate})`

- SBL/SME: band lookup on **recommended** amount. Boundary rule to code explicitly (PRD bands overlap at ₦5m/₦10m edges — **decision:** lower-bound-inclusive, upper-bound-exclusive, except top band open-ended; document and test: ₦5,000,000 → first band; ₦5,000,001 → second).
- Agro/Clean/Housing-Edu: fixed rows regardless of amount/status.
- Asset: Admin-configured row required; else block with alert.
- Returning-client definition enforced: repaid facility in good standing OR performing facility with ≥6 months clean history (checkbox evidence + audit).

### 7.4 Stage 11 — Final recommendation (§15)

- Structure: Decision, Recommended Amount, **Recommended Tenor**, **Applicable Lending Rate (% + RB/Flat + basis)**, Repayment structure, Key reasons/risks/mitigations, Conditions.
- §17.4 schedule preview embedded (instalment, P&I split, total interest, EAR).
- **Override (§16):** analyst may change decision/amount/tenor (reason mandatory); **rate** change creates pending-approval state for Admin. System vs Analyst recommendation shown side-by-side with reason.
- **Acceptance:** worked example — Requested ₦5m → Recommended ₦3.2m, 12 mo, Returning SBL 4.60% RB — reproduces instalment/total-interest figures from engine tests; override without reason is rejected; rate override without Admin stays pending.

---

## Phase 8 — Stakeholder Views, Summaries, Export (§18, §21)

1. **Executive Summary** (decision-makers): borrower, product, requested vs recommended, decision, risk level, key reasons, conditions, analyst recommendation + schedule headline.
2. **Detailed Analysis** (credit professionals): all metrics, credit/business/collateral/product panels, evidence links, flags, comments.
3. **Credit Assessment Summary** (§21, 18 items incl. rate + schedule preview) → printable + PDF export with version stamp and rate-basis footnote.
4. **Acceptance:** PDF contains all 18 items; historical PDFs immutable after decision.

---

## Phase 9 — Alerts, Audit, Analytics (§20 + Admin)

- Alert engine runs on every save: missing info, inconsistencies, high DTI / weak DSCR, poor history, excessive exposure, unverified docs, collateral/doc concerns, performance deviation. Each alert: `{code, severity, whyItMatters, fixLink}`.
- Audit log viewer (filter by assessment/actor/action) + export.
- Portfolio-lite analytics (MVP): counts by decision/product, override rate, median recommended-vs-requested haircut. (Full portfolio mgmt explicitly out of scope.)
- **Acceptance:** triggering fixtures raise the right alerts; every override/rate change has an audit row.

---

## Phase 10 — Hardening, Testing, Deployment, Docs

1. **Testing:** engine unit tests (target ≥90%), API integration tests (rate resolution, override flow, immutability), Playwright E2E (one happy-path per product + one decline + one override), accessibility pass (keyboard, contrast, focus order on stepper).
2. **Security:** RBAC tests, rate-table tamper tests (analyst cannot write), file-type/size limits, virus-scan hook point, PII minimisation, session expiry.
3. **Data:** backup/restore runbook for Postgres; MinIO→S3 migration notes; seed versioning.
4. **Deploy:** `Dockerfile` (multi-stage) + `compose.prod.yml`; GitHub Actions build/push image; staging → prod promotion checklist.
5. **Docs:** analyst user guide (11 stages with screenshots), admin guide (rates/thresholds/effective dating), engine math appendix (RB vs Flat formulas + EAR), ADRs (stack, kobo-money, effective-dated rates, no-auto-accept docs).
6. **Acceptance & success mapping (§23):** demo script proving all 8 success criteria, esp. #6 (explicit recommendation with rate/tenor/schedule) and #7 (correct automatic pricing).

---

## Appendix A — Build Order & Dependencies (Gantt-style)

```
P0 Foundations ─┐
P1 Design shell ─┤
P2 Data/Auth/Admin ├─→ P3 Intake/Profile/Docs ─→ P4 Financial engine ─→ P5 Risk engine
                                              └→ P6 Qual/Collateral/Product ─→ P7 Recommend/Price/Override ─→ P8 Summaries/PDF ─→ P9 Alerts/Audit ─→ P10 Harden/Deploy
```

- P4 (engine) can start in parallel with P3 once schemas land.
- P7 is the integration milestone — do not start P8 until the ₦5m→₦3.2m worked example passes.

## Appendix B — Key Decisions Log (to confirm)

1. Six products (not five) — fix PRD wording.
2. SBL/SME band boundaries: lower-inclusive/upper-exclusive.
3. Returning-client evidence requirements (§17 rule adopted).
4. Asset-loan rate: blocked until Admin configures (no silent default).
5. Assisted extraction: MVP = upload + manual entry + verification; OCR/AI deferred (PRD allows "where appropriate").
6. Branch: keep `master` or rename to `main` (recommend `main` to match GitHub default).

## Appendix C — Risks & Mitigations

| Risk | Mitigation |
|---|---|
| Rate mispricing from band-edge bugs | Property-based tests on boundaries + dry-run preview in Admin |
| Float rounding on schedules | Kobo integers + `decimal.js`, totals asserted in tests |
| Analysts trusting unverified docs | Verification gate blocks recommendation; audit on every verify |
| Scope creep into servicing/collections | Hard MVP exclusion list in PRD §22; phase gates |
| Flat vs RB confusion for borrowers | `RateBadge` + schedule + EAR always shown together |
