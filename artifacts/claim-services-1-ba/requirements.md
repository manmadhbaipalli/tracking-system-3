# Business Requirements Document
## Integrated Policy, Claims, and Payments Platform
**Task:** claim-services-1 | **Phase:** BA | **Date:** 2026-02-24

---

## Table of Contents
1. [User Stories](#user-stories)
2. [Business Rules](#business-rules)
3. [Non-Functional Requirements](#non-functional-requirements)
4. [Domain Model](#domain-model)
5. [Integration Requirements](#integration-requirements)
6. [Error Handling](#error-handling)
7. [Traceability Matrix](#traceability-matrix)
8. [Open Questions](#open-questions)

---

## 1. User Stories

### 1.1 Access & Authentication

---

#### US-001: Role-Based Access Control

**As an** administrator
**I want to** configure role-based access permissions for policy, claims, and payment modules
**So that** only authorized users can access sensitive insurance data and perform actions within their authority

**Priority:** High | **Story Points:** 5

**Acceptance Criteria:**
- AC-001.1: System enforces role-based access control (RBAC) with roles: Agent, Underwriter, Claims Adjuster, Claims Manager, Payment Processor, Admin
- AC-001.2: Each role has defined permissions per module (Policy, Claims, Payments): Read, Write, Approve, Admin
- AC-001.3: Unauthorized access attempts return HTTP 403 with message "Access denied"
- AC-001.4: Session tokens expire after configurable idle timeout
- AC-001.5: All login/logout events are recorded in audit log with user ID and timestamp

**Error Scenarios:**
- E-001.1: Invalid credentials → 401 "Authentication failed"
- E-001.2: Expired session → 401 "Session expired, please login again"
- E-001.3: Insufficient permissions → 403 "You do not have permission to perform this action"

---

#### US-002: Audit Logging

**As a** compliance officer
**I want to** have all user actions automatically logged with user ID and timestamp
**So that** we maintain a complete audit trail for regulatory compliance and forensic investigation

**Priority:** High | **Story Points:** 5

**Acceptance Criteria:**
- AC-002.1: Every action (search, create, edit, delete, payment, claim filing) is logged
- AC-002.2: Audit log entry contains: user_id, action_type, entity_type, entity_id, timestamp, before_state (JSON), after_state (JSON)
- AC-002.3: Audit logs are immutable — no delete or update operations permitted
- AC-002.4: Audit logs are retained for minimum 7 years
- AC-002.5: Admins can search and filter audit logs by user, entity, action type, and date range

**Error Scenarios:**
- E-002.1: Audit log write failure → Transaction rolls back, action not applied, error surfaced to user
- E-002.2: Audit log query timeout → 504 "Audit log search timed out, please narrow your criteria"

---

### 1.2 Policy Management

---

#### US-003: Policy Search

**As an** insurance agent
**I want to** search for policies using multiple criteria
**So that** I can quickly locate the correct policy for a customer inquiry or claim

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-003.1: Search criteria supported: Policy Number, Insured First Name, Insured Last Name, Policy Type, Loss Date, Policy City, Policy State, Policy Zip Code, SSN/TIN, Organizational Name
- AC-003.2: Search supports both exact match and partial match (LIKE/contains) for: name fields, policy number, address fields
- AC-003.3: Multiple search criteria can be combined (AND logic)
- AC-003.4: Results are returned within 3 seconds
- AC-003.5: Results display: Insured Name, Policy Number, Policy Type, Effective Date, Expiration Date, Policy Status
- AC-003.6: SSN/TIN is masked in search results (display last 4 digits only: XXX-XX-1234)
- AC-003.7: A "Reset" button clears all search criteria to empty/default values
- AC-003.8: If no results found, display: "No matching policies found."

**Error Scenarios:**
- E-003.1: Search with no criteria → 400 "Please enter at least one search criterion"
- E-003.2: System unavailable → 503 "System is currently unavailable."
- E-003.3: Search timeout (>3s) → 504 "Search timed out. Please refine your criteria."

---

#### US-004: Policy Detail View

**As an** insurance agent
**I want to** view full policy details for a selected policy
**So that** I can review coverage, insured details, and policy status to assist a customer

**Priority:** High | **Story Points:** 5

**Acceptance Criteria:**
- AC-004.1: Policy detail displays: Insured Name, Policy Number, Policy Type, Effective Date, Expiration Date, Policy Status
- AC-004.2: Vehicle details displayed when policy type is AUTO: Year, Make, Model, VIN
- AC-004.3: Location/Property details displayed: Address Line 1, Address Line 2, City, State, Zip Code
- AC-004.4: Coverage Details displayed for each coverage: Coverage Type, Limit Amount, Deductible Amount
- AC-004.5: SSN/TIN is masked (last 4 digits only)
- AC-004.6: Policy detail page is WCAG 2.1 AA compliant (keyboard navigation, screen reader support, color contrast)
- AC-004.7: Policy details load within 5 seconds
- AC-004.8: All views of policy detail are recorded in audit log

**Error Scenarios:**
- E-004.1: Policy ID not found → 404 "Policy not found"
- E-004.2: Detail retrieval failure → 503 "Unable to retrieve details. Please try again later."

---

#### US-005: Policy Creation

**As an** underwriter
**I want to** create a new insurance policy with coverage selections
**So that** I can bind coverage for a qualified insured

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-005.1: System generates unique policy number in format: POL-YYYY-NNNNNN (auto-sequential)
- AC-005.2: Required fields: Insured, Policy Type, Effective Date, Expiration Date, at least one Coverage
- AC-005.3: Effective date must be today or a future date (POL-002)
- AC-005.4: Expiration date must be after effective date
- AC-005.5: At least one Coverage with limit and deductible is required (COV-004)
- AC-005.6: Policy status set to QUOTED on creation; transitions to ACTIVE after binding
- AC-005.7: Total premium is calculated and displayed based on rating engine rules (PRM-001)
- AC-005.8: Creation action is recorded in audit log

**Error Scenarios:**
- E-005.1: Effective date in the past → 400 "Effective date must be today or a future date"
- E-005.2: Expiration date before effective date → 400 "Expiration date must be after effective date"
- E-005.3: No coverage selected → 400 "At least one coverage is required"
- E-005.4: Coverage limit exceeds aggregate limit → 400 "Coverage limit cannot exceed policy aggregate limit"

---

#### US-006: Policy Update / Endorsement

**As an** underwriter
**I want to** modify an active policy (endorsement)
**So that** I can accommodate mid-term changes requested by the insured

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-006.1: Endorsement generates unique endorsement number in format: END-YYYY-NNNNNN
- AC-006.2: Endorsement types supported: ADD_COVERAGE, REMOVE_COVERAGE, CHANGE_LIMIT, CHANGE_DEDUCTIBLE, ADD_VEHICLE, REMOVE_VEHICLE, ADDRESS_CHANGE
- AC-006.3: Endorsement effective date must be within the policy period
- AC-006.4: Premium change (increase or decrease) is calculated and displayed before confirmation
- AC-006.5: System records before/after state in audit log
- AC-006.6: Policy status remains ACTIVE during and after endorsement
- AC-006.7: Endorsement is linked to parent policy and visible in policy detail

**Error Scenarios:**
- E-006.1: Endorsement effective date outside policy period → 400 "Endorsement date must be within policy effective/expiration period"
- E-006.2: Removing last coverage → 400 "Policy must retain at least one coverage"

---

#### US-007: Policy Cancellation

**As an** underwriter
**I want to** cancel an active policy
**So that** I can terminate coverage and process any applicable refunds

**Priority:** Medium | **Story Points:** 5

**Acceptance Criteria:**
- AC-007.1: Policy can be cancelled with a specified cancellation effective date
- AC-007.2: Cancellation notice must be at minimum 30 days (POL-004)
- AC-007.3: System calculates pro-rata refund based on unused premium (POL-005)
- AC-007.4: Cancellation reason code is required (INSURED_REQUEST, NON_PAYMENT, UNDERWRITING, FRAUD, OTHER)
- AC-007.5: Policy status transitions to CANCELLED
- AC-007.6: Cancellation is recorded in audit log
- AC-007.7: Open claims on the policy are flagged for review

**Error Scenarios:**
- E-007.1: Cancellation notice < 30 days → 400 "Minimum 30-day cancellation notice required"
- E-007.2: Cancelling already cancelled policy → 409 "Policy is already cancelled"

---

### 1.3 Claims Management

---

#### US-008: Claim Search

**As a** claims adjuster
**I want to** search for claims by multiple criteria
**So that** I can locate a claim quickly for investigation or customer inquiry

**Priority:** High | **Story Points:** 5

**Acceptance Criteria:**
- AC-008.1: Search criteria: Claim Number, Policy Number, Insured Name, Date of Loss (range), Claim Status
- AC-008.2: Claim Status filter values: Open, Closed, Paid, Denied
- AC-008.3: Results display: Claim Number, Date of Loss, Claim Status, Insured Name, Policy Number
- AC-008.4: Results sorted by Date of Loss descending (most recent first)
- AC-008.5: Results returned within 3 seconds
- AC-008.6: If no results, display: "No matching claims found."

**Error Scenarios:**
- E-008.1: System unavailable → 503 "System is currently unavailable."
- E-008.2: Search timeout → 504 "Search timed out. Please refine your criteria."

---

#### US-009: FNOL — First Notice of Loss

**As a** policyholder or claims adjuster
**I want to** file a First Notice of Loss (FNOL) claim against an active policy
**So that** I can initiate the claims process for a covered loss event

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-009.1: System verifies the policy is ACTIVE on the date of loss before accepting FNOL (CLM-002)
- AC-009.2: System verifies the loss type matches a coverage on the policy (CLM-003)
- AC-009.3: System assigns claim number in format: CLM-YYYY-NNNNNN (auto-sequential) (CLM-001)
- AC-009.4: Required FNOL fields: Date of Loss, Loss Type, Loss Description, Estimated Amount, Claimant Information
- AC-009.5: Initial reserve set to estimated amount (CLM-004)
- AC-009.6: Claim status set to FNOL on creation
- AC-009.7: If estimated amount exceeds coverage limit, system displays warning (not a hard error)
- AC-009.8: FNOL submission recorded in audit log

**Error Scenarios:**
- E-009.1: Policy not active on date of loss → 400 "Policy not active for date of loss"
- E-009.2: Loss type not covered by policy → 400 "Coverage type not found on policy"
- E-009.3: Date of loss in the future → 400 "Date of loss cannot be in the future"
- E-009.4: Missing required fields → 400 with field-level validation messages

---

#### US-010: Claim Detail View & History

**As a** claims adjuster
**I want to** view full claim details and the complete claim history for a policy
**So that** I can understand the claim context and make informed adjudication decisions

**Priority:** High | **Story Points:** 5

**Acceptance Criteria:**
- AC-010.1: Claim detail displays: Claim Number, Policy Number, Insured Name, Date of Loss, Reported Date, Loss Type, Loss Description, Claim Status, Reserve Amount, Paid Amount
- AC-010.2: Claim History section shows all prior claims on the policy: Claim Number, Date of Loss, Claim Status
- AC-010.3: Claim history sorted by Date of Loss descending
- AC-010.4: Claim history filterable by Claim Status (Open, Closed, Paid, Denied)
- AC-010.5: Claim detail and history load within 5 seconds
- AC-010.6: If no prior claims exist, display: "No prior claims exist for this policy."
- AC-010.7: View is WCAG 2.1 AA compliant

**Error Scenarios:**
- E-010.1: Claim ID not found → 404 "Claim not found"
- E-010.2: Unable to load history → 503 "Unable to retrieve details. Please try again later."

---

#### US-011: Claim Investigation & Adjudication

**As a** claims adjuster
**I want to** update claim status through investigation and adjustment workflow
**So that** I can move a claim from FNOL to settlement or denial

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-011.1: Claim status transitions supported: FNOL → UNDER_REVIEW → APPROVED → SETTLED | DENIED → CLOSED
- AC-011.2: Adjuster can update reserve amount during investigation; each change is audit-logged with reason
- AC-011.3: Adjuster can record investigation notes and attach documents
- AC-011.4: Denial requires mandatory denial reason code: NOT_COVERED, POLICY_LAPSED, FRAUD, LATE_FILING, DUPLICATE, OTHER
- AC-011.5: Settlement amount cannot exceed reserve amount without manager approval (CLM-005)
- AC-011.6: Claim can be reopened from CLOSED status with justification note
- AC-011.7: All status changes recorded in audit log with user, timestamp, reason

**Error Scenarios:**
- E-011.1: Invalid status transition → 400 "Invalid status transition from [current] to [target]"
- E-011.2: Settlement exceeds reserve without approval → 403 "Settlement exceeds reserve. Manager approval required."

---

#### US-012: Claim-Level Policy Data Override

**As a** claims adjuster
**I want to** add or edit policy information at the claim level when the associated policy is unverified
**So that** the claim can proceed without altering the original policy record

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-012.1: Claim-level policy data fields are editable when policy status is UNVERIFIED or when policy is not found
- AC-012.2: Changes to claim-level policy data are stored separately and do NOT overwrite original policy records
- AC-012.3: UI displays a visual indicator (banner/badge) when claim is using claim-level policy data instead of verified policy data
- AC-012.4: All changes to claim-level policy data are audit-logged: user, timestamp, field changed, before/after values
- AC-012.5: If claim-level policy data cannot be saved → display: "Unable to save claim-level policy data. Please try again later."

**Error Scenarios:**
- E-012.1: Save failure → 503 "Unable to save claim-level policy data. Please try again later."
- E-012.2: Attempted overwrite of verified policy data → 409 "Cannot overwrite verified policy data at claim level"

---

#### US-013: Subrogation Referral

**As a** claims manager
**I want to** refer a settled claim to the subrogation team
**So that** we can pursue recovery from the responsible third party

**Priority:** Medium | **Story Points:** 5

**Acceptance Criteria:**
- AC-013.1: Claims with SETTLED or CLOSED status can be referred to subrogation
- AC-013.2: Subrogation referral captures: responsible party details, estimated recovery amount, referral notes
- AC-013.3: Claim record shows subrogation referral status and recovery amount
- AC-013.4: Other Carrier Information captured: carrier name, policy number, contact, coverage details
- AC-013.5: Referral recorded in audit log

**Error Scenarios:**
- E-013.1: Referring a DENIED claim → 400 "Cannot refer denied claim to subrogation"

---

#### US-014: Injury Incident & Coding Details

**As a** claims adjuster
**I want to** record injury incident details and coding information on a claim
**So that** medical claims can be processed accurately with correct billing codes

**Priority:** Medium | **Story Points:** 5

**Acceptance Criteria:**
- AC-014.1: Injury fields captured: injury type, body part, severity, treatment facility, treating physician
- AC-014.2: Medical coding fields: ICD-10 diagnosis codes, CPT procedure codes
- AC-014.3: Carrier involvement details captured: other carrier name, policy number, coverage type
- AC-014.4: Multiple injury records supported per claim
- AC-014.5: Coding data stored and accessible for EDI 835/837 processing

**Error Scenarios:**
- E-014.1: Invalid ICD-10 code format → 400 "Invalid ICD-10 code format"
- E-014.2: Invalid CPT code → 400 "Invalid CPT procedure code"

---

### 1.4 Payments & Disbursements

---

#### US-015: Scheduled Payments Management

**As a** claims adjuster
**I want to** manage scheduled payments on a claim
**So that** structured settlements and periodic disbursements are tracked accurately

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-015.1: Scheduled payment fields: Applicability, Payment Type, Total Amount, Balance, Current Amount Due, Recipient Identification
- AC-015.2: Multiple scheduled payment records supported per claim
- AC-015.3: Balance auto-calculates: Balance = Total Amount - Sum of Paid Amounts
- AC-015.4: Payment status tracked: SCHEDULED, PROCESSING, PAID, VOIDED, REVERSED
- AC-015.5: Payments linked to specific reserve lines
- AC-015.6: Recipient identification supports: individual claimant, vendor, attorney, medical provider

**Error Scenarios:**
- E-015.1: Payment amount exceeds reserve line balance → 400 "Payment exceeds available reserve balance"
- E-015.2: Recipient not onboarded → 400 "Recipient must complete payment method verification before payment can be issued"

---

#### US-016: Payment Creation & Processing

**As a** payment processor
**I want to** create and process payments linked to claims and policies
**So that** claimants and vendors receive accurate and timely disbursements

**Priority:** High | **Story Points:** 13

**Acceptance Criteria:**
- AC-016.1: Payment methods supported: ACH, Wire Transfer, Credit/Debit Card, Stripe Connect, Global Payouts
- AC-016.2: Payments can have positive, negative, or zero dollar amounts (for adjustments and corrections)
- AC-016.3: Each payment links to: Claim, Policy, Reserve Line, Payee
- AC-016.4: Payment routing rules applied based on: payee type, payment type, business rules configuration
- AC-016.5: Payment supports multiple payees (joint payees) per transaction
- AC-016.6: Payments designated as eroding or non-eroding against reserve lines
- AC-016.7: Income tax withholding configurable per payment; Tax ID (EIN/SSN) captured for payees
- AC-016.8: Tax reportable flag per payment
- AC-016.9: Documents can be attached to payment transactions
- AC-016.10: Payment processing completed within 5 seconds (for digital methods)
- AC-016.11: All payment actions recorded in audit log

**Error Scenarios:**
- E-016.1: Insufficient reserve balance → 400 "Insufficient reserve balance for payment"
- E-016.2: Invalid banking details for ACH/Wire → 400 "Invalid banking information provided"
- E-016.3: Payee KYC not verified → 400 "Payee identity verification required before payment can be issued"
- E-016.4: Payment service unavailable → 503 "Payment processing is currently unavailable. Please try again later."

---

#### US-017: Payment Lifecycle — Void, Reversal, Reissue

**As a** payment processor
**I want to** void, reverse, or reissue payments
**So that** I can correct errors and manage payment exceptions

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-017.1: Void supported for payments in PENDING or PROCESSING status
- AC-017.2: Reversal supported for payments in PAID status; reversal restores reserve line balance
- AC-017.3: Reissue creates a new payment record referencing the original voided/reversed payment
- AC-017.4: All void/reversal/reissue actions require reason code and manager approval above configurable threshold
- AC-017.5: Reserve line balances update in real-time on void/reversal
- AC-017.6: Idempotency key enforced to prevent duplicate payments

**Error Scenarios:**
- E-017.1: Void attempted on PAID payment → 400 "Cannot void a paid payment; use reversal"
- E-017.2: Reversal on already-reversed payment → 409 "Payment has already been reversed"

---

#### US-018: Vendor & Claimant Onboarding with KYC

**As a** payment processor
**I want to** onboard vendors and claimants with secure payment method verification including KYC
**So that** disbursements are made only to verified, legitimate recipients

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-018.1: Onboarding captures: entity type (individual/organization), legal name, Tax ID (SSN/EIN), contact information, banking details or preferred payment method
- AC-018.2: KYC identity verification integrated with third-party identity service
- AC-018.3: Onboarding status tracked: PENDING, VERIFIED, REJECTED, SUSPENDED
- AC-018.4: Payments cannot be issued to PENDING or REJECTED payees
- AC-018.5: Banking details encrypted at rest and in transit (PCI-DSS)
- AC-018.6: Tax ID masked in all display contexts (last 4 digits only)

**Error Scenarios:**
- E-018.1: KYC verification fails → 422 "Identity verification failed. Please review submitted documents."
- E-018.2: Duplicate Tax ID detected → 409 "A payee with this Tax ID already exists"

---

#### US-019: Automated Payment Line Items from Estimates

**As a** claims adjuster
**I want to** automatically create payable line items from Xactimate/XactAnalysis estimates
**So that** property damage payments are generated accurately without manual data entry

**Priority:** Medium | **Story Points:** 8

**Acceptance Criteria:**
- AC-019.1: System receives estimate data from Xactimate/XactAnalysis via integration
- AC-019.2: Each line item in estimate maps to a payable line item on the claim
- AC-019.3: Total estimate amount validated against coverage limit before creating payments
- AC-019.4: Adjuster can review, approve, or reject individual line items before payment is created
- AC-019.5: Approved line items create payment records automatically
- AC-019.6: Integration supports error handling and retry logic for failed imports

**Error Scenarios:**
- E-019.1: Estimate total exceeds coverage limit → Warning flagged; adjuster must confirm override
- E-019.2: Integration unavailable → 503 "Estimate import service unavailable. Please try again later."

---

#### US-020: EDI/EOB Remittance for Medical Payments

**As a** payment processor
**I want to** send EDI 835 remittance advice and process EDI 837 claim submissions for medical providers
**So that** medical providers receive accurate payment explanations and billing is processed electronically

**Priority:** Medium | **Story Points:** 13

**Acceptance Criteria:**
- AC-020.1: EDI 837 (claim submission) ingestion supported from medical providers
- AC-020.2: EDI 835 (remittance advice) generated and transmitted to medical providers on payment
- AC-020.3: CPT/ICD code mappings validated against standard code sets
- AC-020.4: Bill review vendor integration for adjudication and adjustment of medical bills
- AC-020.5: Explanation of Benefits (EOB) style output generated for each medical payment
- AC-020.6: Adjustments (reductions, contractual write-offs) captured with reason codes

**Error Scenarios:**
- E-020.1: Invalid EDI 837 format → 400 "EDI 837 format validation failed: [details]"
- E-020.2: Bill review service unavailable → 503 "Bill review service unavailable. Please retry later."

---

#### US-021: Negotiation & Settlement Management

**As a** claims manager
**I want to** manage negotiation workflows, coverage opinion requests, and settlement plans
**So that** complex claims are resolved with proper documentation and tracking

**Priority:** Medium | **Story Points:** 8

**Acceptance Criteria:**
- AC-021.1: Negotiation records capture: negotiation type, opening demand, counter offers, final settlement
- AC-021.2: Coverage opinion request workflow: request, response, coverage determination recorded
- AC-021.3: Settlement plan tracks: total settlement amount, payment schedule, signed agreement status
- AC-021.4: Settlement plan linked to structured payment schedule (US-015)
- AC-021.5: All negotiation events recorded with user, timestamp, and amounts

**Error Scenarios:**
- E-021.1: Settlement plan total exceeds approved reserve → 400 "Settlement plan total exceeds approved reserve amount"

---

#### US-022: Reserve Management

**As a** claims adjuster
**I want to** manage reserve lines for claims
**So that** financial liability is accurately tracked and reported

**Priority:** High | **Story Points:** 8

**Acceptance Criteria:**
- AC-022.1: Multiple reserve lines per claim supported (e.g., Medical, Indemnity, Expense, Legal)
- AC-022.2: Reserve lines track: reserve type, initial amount, current amount, paid-to-date
- AC-022.3: Payments allocated across multiple reserve lines
- AC-022.4: Reserve changes require reason code and are audit-logged
- AC-022.5: Reserve cannot be reduced below paid-to-date amount

**Error Scenarios:**
- E-022.1: Reserve reduction below paid amount → 400 "Reserve cannot be less than amount already paid"

---

---

## 2. Business Rules

### 2.1 Policy Rules

| Rule ID | Description |
|---------|-------------|
| POL-001 | Policy number format: POL-YYYY-NNNNNN (auto-generated, sequential) |
| POL-002 | Effective date must be >= today for new policies |
| POL-003 | Expiration date must be > effective date |
| POL-004 | Policy cancellation requires minimum 30 days written notice |
| POL-005 | Cancellation refund calculated pro-rata based on unused premium days |
| POL-006 | Policy term length: 6 months or 12 months |
| POL-007 | Policy status transitions: QUOTED → BOUND → ISSUED → ACTIVE → (CANCELLED | EXPIRED) |
| POL-008 | Endorsement effective date must be within policy effective-to-expiration period |
| POL-009 | Endorsement number format: END-YYYY-NNNNNN (auto-generated) |
| POL-010 | SSN/TIN displayed as masked (format: XXX-XX-1234) everywhere except authorized admin views |
| POL-011 | Policy records must be retained for minimum 7 years post-expiration |
| POL-012 | Policy search returns results within 3 seconds |

### 2.2 Coverage Rules

| Rule ID | Description |
|---------|-------------|
| COV-001 | Each coverage requires a limit amount (positive decimal) |
| COV-002 | Deductible amount must be <= limit amount |
| COV-003 | Sum of all coverage limits on a policy must not exceed the policy aggregate limit |
| COV-004 | Policy must have at least one active coverage |
| COV-005 | Supported coverage types: LIABILITY, COLLISION, COMPREHENSIVE, MEDICAL_PAYMENTS, UNINSURED_MOTORIST, PROPERTY_DAMAGE, BODILY_INJURY, UMBRELLA |
| COV-006 | Coverage can be added or removed only via endorsement on active policies |

### 2.3 Premium Rules

| Rule ID | Description |
|---------|-------------|
| PRM-001 | Base Premium Formula: Base Rate × Territory Factor × Risk Factor × Coverage Factor - Discounts + Surcharges |
| PRM-002 | Multi-policy discount: 10% if insured has 2+ active policies |
| PRM-003 | Claims-free discount: 5% for 3+ consecutive years without claims |
| PRM-004 | Minimum premium per term: $100 |
| PRM-005 | Premium must be recalculated on every endorsement; premium change (+ or -) displayed before confirmation |
| PRM-006 | Pro-rata refund on cancellation: Refund = (Remaining Days / Total Policy Days) × Total Premium Paid |

### 2.4 Claim Rules

| Rule ID | Description |
|---------|-------------|
| CLM-001 | Claim number format: CLM-YYYY-NNNNNN (auto-generated, sequential) |
| CLM-002 | Date of loss must fall within policy effective date and expiration date (inclusive) |
| CLM-003 | Claimed coverage type must match at least one coverage on the policy |
| CLM-004 | Initial reserve amount = estimated loss amount provided at FNOL |
| CLM-005 | Settlement amount must not exceed current reserve; override requires manager approval |
| CLM-006 | Claim status transitions: FNOL → UNDER_REVIEW → APPROVED → SETTLED | DENIED → CLOSED |
| CLM-007 | Claims can be reopened from CLOSED status with mandatory justification note |
| CLM-008 | Claims sorted by Date of Loss descending in list views |
| CLM-009 | Open claims must be flagged when associated policy is cancelled |
| CLM-010 | Claim-level policy data changes stored separately; original policy data NOT overwritten |
| CLM-011 | Subrogation referrals allowed only for SETTLED or CLOSED claims |
| CLM-012 | Reserve cannot be reduced below total paid-to-date for that reserve line |

### 2.5 Payment Rules

| Rule ID | Description |
|---------|-------------|
| PAY-001 | Payments must be linked to a claim and policy |
| PAY-002 | Payments can be positive, negative, or zero dollar amounts |
| PAY-003 | Payments require verified payee (KYC completed) before disbursement |
| PAY-004 | Idempotency key enforced to prevent duplicate payment submissions |
| PAY-005 | Eroding payments reduce the coverage limit; non-eroding payments do not |
| PAY-006 | Reserve line balance updated in real-time on payment creation, void, or reversal |
| PAY-007 | Joint payments (multiple payees) must identify all payees and their respective portions |
| PAY-008 | Tax reportable payments must capture payee Tax ID (SSN/EIN) |
| PAY-009 | All payment data encrypted at rest and in transit (PCI-DSS) |
| PAY-010 | Payment processing must be idempotent (retry-safe) |
| PAY-011 | Void allowed only for PENDING or PROCESSING payments; reversal for PAID payments |
| PAY-012 | Payment routing rules configurable by: payee type, payment type, business rules |

### 2.6 Policy Lifecycle State Machine

```
QUOTED ──bind──► BOUND ──issue──► ISSUED ──activate──► ACTIVE
                                                           │
                                        ┌──────────────────┼──────────────────┐
                                        │                  │                  │
                                   endorse             cancel             expire
                                        │                  │                  │
                                  (stays ACTIVE)      CANCELLED           EXPIRED
                                        │
                                     renew
                                        │
                                     ACTIVE (new term)
```

### 2.7 Claim Lifecycle State Machine

```
FNOL ──review──► UNDER_REVIEW ──adjust──► APPROVED ──settle──► SETTLED ──close──► CLOSED
                                               │                                      │
                                            deny                                   reopen
                                               │                                      │
                                            DENIED ──close──► CLOSED          UNDER_REVIEW
```

---

## 3. Non-Functional Requirements

### 3.1 Performance

| NFR-P | Requirement |
|-------|-------------|
| NFR-P-001 | Policy search results returned within 3 seconds |
| NFR-P-002 | Policy detail and claim history loaded within 5 seconds |
| NFR-P-003 | Payment processing completed within 5 seconds for digital methods |
| NFR-P-004 | Premium calculation completed within 2 seconds |
| NFR-P-005 | API response time < 500ms for CRUD operations; < 2 seconds for calculation endpoints |
| NFR-P-006 | System supports concurrent user sessions without performance degradation |

### 3.2 Security

| NFR-S | Requirement |
|-------|-------------|
| NFR-S-001 | SSN/TIN encrypted at rest using AES-256; masked in all UI displays (last 4 digits) |
| NFR-S-002 | Payment data (card numbers, banking details) encrypted at rest and in transit (TLS 1.2+) |
| NFR-S-003 | PCI-DSS compliance for all payment data handling |
| NFR-S-004 | Role-based access control enforced for all modules (Agent, Underwriter, Claims Adjuster, Claims Manager, Payment Processor, Admin) |
| NFR-S-005 | All sensitive data masked in application logs (SSN, payment info, Tax ID) |
| NFR-S-006 | Comprehensive audit logs for all user actions: create, read (policy/claim detail), update, delete, payment actions |
| NFR-S-007 | Audit logs are immutable and retained for minimum 7 years |
| NFR-S-008 | Multi-factor authentication (MFA) for all users with payment processing access |
| NFR-S-009 | Session idle timeout enforced; configurable per role |

### 3.3 Regulatory Compliance

| NFR-R | Requirement |
|-------|-------------|
| NFR-R-001 | Policy records retained for minimum 7 years post-expiration |
| NFR-R-002 | Audit logs retained for minimum 7 years |
| NFR-R-003 | Minimum 30-day written cancellation notice enforced by system |
| NFR-R-004 | State-specific data privacy compliance supported (configurable per jurisdiction) |
| NFR-R-005 | Annual premium reports and claims reports supported for regulatory filing |
| NFR-R-006 | Tax reporting: 1099 support for payments to reportable payees |
| NFR-R-007 | EDI 835/837 compliance for medical payment transactions |
| NFR-R-008 | WCAG 2.1 AA accessibility compliance for all UI interfaces |

### 3.4 Reliability

| NFR-RL | Requirement |
|--------|-------------|
| NFR-RL-001 | 99.9% uptime for policy and claims services (max ~8.7 hours downtime/year) |
| NFR-RL-002 | Transaction integrity for all payment operations (ACID compliance) |
| NFR-RL-003 | Idempotent payment API operations — safe to retry without duplicate disbursements |
| NFR-RL-004 | All external integrations must implement error handling and retry logic with exponential backoff |
| NFR-RL-005 | Integration failures must not block core policy/claim workflows; graceful degradation required |

### 3.5 Scalability & Maintainability

| NFR-SM | Requirement |
|--------|-------------|
| NFR-SM-001 | ReactJS (frontend) / Python FastAPI (backend) technology stack |
| NFR-SM-002 | Payment routing rules configurable via admin UI without code changes |
| NFR-SM-003 | API-first design; all business operations exposed via REST APIs |

---

## 4. Domain Model (ACORD Entity Hierarchy)

### 4.1 Entity Relationship Overview

```
Insured (Individual / Organization)
└── Policy
    ├── Coverage (1:N)
    │   ├── Limit
    │   └── Deductible
    ├── Premium
    │   ├── Base Premium
    │   ├── Rating Factors
    │   ├── Discounts
    │   └── Surcharges
    ├── Endorsement (0:N)
    ├── Vehicle (0:N) — AUTO policies
    └── Claim (0:N)
        ├── Claimant
        ├── Reserve (1:N)
        │   └── Reserve Line
        ├── ClaimLevelPolicyData (override, 0:1)
        ├── InjuryIncident (0:N)
        ├── SubrogationReferral (0:1)
        ├── NegotiationRecord (0:N)
        └── Payment (0:N)
            ├── PaymentDetail (Payee, Amount Portion)
            ├── PaymentMethod (ACH, Wire, Card, Stripe)
            └── RemittanceAdvice (medical payments)

Payee (Vendor / Claimant / Attorney / MedicalProvider)
└── PaymentMethod (KYC Verified)

PaymentRoutingRule
AuditLog
```

### 4.2 Entity Attribute Tables

#### Insured
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK, auto-gen | Unique identifier |
| type | Enum | INDIVIDUAL, ORGANIZATION | Insured entity type |
| first_name | String(100) | Required for INDIVIDUAL | First name |
| last_name | String(100) | Required for INDIVIDUAL | Last name |
| organization_name | String(200) | Required for ORGANIZATION | Legal business name |
| date_of_birth | Date | Required for INDIVIDUAL | DOB for rating |
| ssn_tin | String(encrypted) | Unique, masked in UI | SSN (individual) or TIN (org) |
| email | String(255) | Unique | Contact email |
| phone | String(20) | | Contact phone |
| address_line1 | String(255) | Required | Street address |
| address_line2 | String(255) | Optional | Suite/Apt |
| city | String(100) | Required | City |
| state | String(2) | Required | State code |
| zip_code | String(10) | Required | ZIP/ZIP+4 |
| created_at | DateTime | Auto | Record creation timestamp |
| updated_at | DateTime | Auto | Last update timestamp |

#### Policy
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK, auto-gen | Unique identifier |
| policy_number | String(20) | Unique, POL-YYYY-NNNNNN | Auto-generated policy number |
| insured_id | Long | FK → Insured | Policy owner |
| policy_type | Enum | AUTO, HOME, LIFE, HEALTH, COMMERCIAL | Line of business |
| status | Enum | QUOTED, BOUND, ISSUED, ACTIVE, CANCELLED, EXPIRED | Policy lifecycle status |
| effective_date | Date | >= today on creation | Coverage start |
| expiration_date | Date | > effective_date | Coverage end |
| aggregate_limit | Decimal | > 0 | Maximum total payout |
| total_premium | Decimal | Calculated | Computed by rating engine |
| cancellation_date | Date | Optional | Effective cancellation date |
| cancellation_reason | Enum | Optional | INSURED_REQUEST, NON_PAYMENT, UNDERWRITING, FRAUD, OTHER |
| refund_amount | Decimal | Optional | Pro-rata refund on cancellation |
| created_at | DateTime | Auto | |
| updated_at | DateTime | Auto | |

#### Coverage
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| policy_id | Long | FK → Policy | Parent policy |
| coverage_type | Enum | LIABILITY, COLLISION, COMPREHENSIVE, MEDICAL_PAYMENTS, UNINSURED_MOTORIST, PROPERTY_DAMAGE, BODILY_INJURY, UMBRELLA | Coverage type |
| coverage_code | String(20) | ACORD standard | ACORD coverage code |
| limit_amount | Decimal | > 0 | Per-occurrence limit |
| deductible_amount | Decimal | >= 0, <= limit | Out-of-pocket |
| premium_portion | Decimal | Calculated | Premium attributable to this coverage |
| is_active | Boolean | Default true | Active/inactive flag |
| effective_date | Date | | Coverage start (can differ from policy) |
| expiration_date | Date | | Coverage end |

#### Premium
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| policy_id | Long | FK → Policy | Parent policy |
| base_rate | Decimal | | Base rate before factors |
| territory_factor | Decimal | | Geographic rating factor |
| risk_factor | Decimal | | Risk classification factor |
| coverage_factor | Decimal | | Coverage type factor |
| multi_policy_discount | Decimal | 0.10 if applicable | 10% discount |
| claims_free_discount | Decimal | 0.05 if applicable | 5% discount |
| surcharges | Decimal | | Additional charges |
| total_premium | Decimal | Calculated | Final premium amount |
| effective_date | Date | | When this premium applies |
| calculated_at | DateTime | Auto | Calculation timestamp |

#### Endorsement
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| endorsement_number | String(20) | Unique, END-YYYY-NNNNNN | Auto-generated |
| policy_id | Long | FK → Policy | Parent policy |
| type | Enum | ADD_COVERAGE, REMOVE_COVERAGE, CHANGE_LIMIT, CHANGE_DEDUCTIBLE, ADD_VEHICLE, REMOVE_VEHICLE, ADDRESS_CHANGE | Change type |
| effective_date | Date | Within policy period | Change effective date |
| premium_change | Decimal | Can be negative | Premium increase/decrease |
| description | Text | | Description of change |
| created_by | Long | FK → User | Who created endorsement |
| created_at | DateTime | Auto | |

#### Vehicle (AUTO policies)
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| policy_id | Long | FK → Policy | Parent policy |
| year | Integer | 4-digit year | Vehicle year |
| make | String(100) | | Manufacturer |
| model | String(100) | | Model name |
| vin | String(17) | Unique | Vehicle Identification Number |
| is_active | Boolean | | Active on policy |

#### Claim
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| claim_number | String(20) | Unique, CLM-YYYY-NNNNNN | Auto-generated |
| policy_id | Long | FK → Policy | Associated policy |
| insured_id | Long | FK → Insured | Claimant's insured record |
| status | Enum | FNOL, UNDER_REVIEW, APPROVED, SETTLED, DENIED, CLOSED | Claim lifecycle |
| date_of_loss | Date | Within policy period | When loss occurred |
| reported_date | DateTime | Auto on FNOL | When claim filed |
| loss_type | Enum | COLLISION, THEFT, FIRE, WATER, LIABILITY, BODILY_INJURY, OTHER | Nature of loss |
| loss_description | Text | Required | Narrative description |
| estimated_amount | Decimal | | Claimant's estimated loss |
| reserve_amount | Decimal | = estimated_amount initially | Current total reserve |
| paid_amount | Decimal | | Total disbursed |
| settlement_amount | Decimal | Optional, <= reserve | Final settlement |
| denial_reason | Enum | Optional | NOT_COVERED, POLICY_LAPSED, FRAUD, LATE_FILING, DUPLICATE, OTHER |
| uses_claim_level_policy_data | Boolean | Default false | Flag for claim-level override |
| subrogation_referred | Boolean | Default false | Referred to subrogation team |
| adjuster_id | Long | FK → User | Assigned adjuster |
| created_at | DateTime | Auto | |
| updated_at | DateTime | Auto | |

#### ClaimLevelPolicyData
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| claim_id | Long | FK → Claim, Unique | One per claim |
| policy_number_override | String | Optional | Claim-level policy number |
| insured_name_override | String | Optional | Claim-level insured name |
| coverage_type_override | Enum | Optional | Claim-level coverage |
| effective_date_override | Date | Optional | Claim-level effective date |
| expiration_date_override | Date | Optional | Claim-level expiration date |
| changed_by | Long | FK → User | Who last modified |
| changed_at | DateTime | Auto | |

#### Reserve (Reserve Line)
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| claim_id | Long | FK → Claim | Parent claim |
| reserve_type | Enum | MEDICAL, INDEMNITY, EXPENSE, LEGAL, OTHER | Reserve category |
| initial_amount | Decimal | | Reserve at FNOL |
| current_amount | Decimal | | Current reserve |
| paid_to_date | Decimal | | Total paid against this reserve |
| is_eroding | Boolean | | Does payment erode coverage limit |
| last_updated_by | Long | FK → User | |
| last_updated_at | DateTime | Auto | |

#### Payment
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| claim_id | Long | FK → Claim | Parent claim |
| policy_id | Long | FK → Policy | Associated policy |
| reserve_id | Long | FK → Reserve | Allocated reserve line |
| payment_method | Enum | ACH, WIRE, CREDIT_CARD, DEBIT_CARD, STRIPE_CONNECT, GLOBAL_PAYOUT | Disbursement method |
| status | Enum | PENDING, PROCESSING, PAID, VOIDED, REVERSED, REISSUED | Payment lifecycle |
| amount | Decimal | Can be negative, zero, or positive | Payment amount |
| is_eroding | Boolean | | Erodes coverage limit |
| is_tax_reportable | Boolean | | 1099 reportable |
| tax_withheld_amount | Decimal | Optional | Income tax withheld |
| idempotency_key | String | Unique | Prevents duplicate payments |
| void_reason | String | If voided | |
| reversal_reason | String | If reversed | |
| original_payment_id | Long | FK → Payment, if reissue | Original payment |
| created_by | Long | FK → User | |
| created_at | DateTime | Auto | |
| processed_at | DateTime | | When payment settled |

#### PaymentDetail (per payee on a payment)
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| payment_id | Long | FK → Payment | Parent payment |
| payee_id | Long | FK → Payee | Recipient |
| amount_portion | Decimal | | This payee's portion |
| is_joint | Boolean | | Part of joint payment |
| deduction_amount | Decimal | Optional | Any deduction applied |
| deduction_reason | String | Optional | |

#### Payee
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| type | Enum | INDIVIDUAL, VENDOR, ATTORNEY, MEDICAL_PROVIDER | Payee category |
| legal_name | String(200) | Required | |
| tax_id | String(encrypted) | Masked in UI | SSN or EIN |
| onboarding_status | Enum | PENDING, VERIFIED, REJECTED, SUSPENDED | KYC status |
| preferred_payment_method | Enum | | Default payment method |
| banking_details | JSON(encrypted) | PCI-DSS | ACH/Wire banking info |
| email | String | | Contact email |
| created_at | DateTime | Auto | |
| verified_at | DateTime | Optional | KYC verification timestamp |

#### AuditLog
| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| id | Long | PK | |
| user_id | Long | FK → User | Actor |
| action_type | Enum | CREATE, READ, UPDATE, DELETE, STATUS_CHANGE, PAYMENT, LOGIN, LOGOUT | Action performed |
| entity_type | String | POLICY, CLAIM, PAYMENT, PAYEE, RESERVE, etc. | Target entity type |
| entity_id | Long | | Target entity ID |
| before_state | JSON | Nullable | State before change |
| after_state | JSON | Nullable | State after change |
| ip_address | String | | Client IP |
| timestamp | DateTime | Auto, immutable | When action occurred |

---

## 5. Integration Requirements

### 5.1 Payment Integrations

| Integration | Direction | Description |
|-------------|-----------|-------------|
| Stripe Connect | Outbound | Card and digital wallet disbursements |
| Global Payouts | Outbound | International payment disbursements |
| Bank ACH | Outbound | ACH electronic fund transfers |
| Bank Wire | Outbound | Domestic and international wire transfers |
| Agency Markets Payment Service | Bidirectional | Agency-specific payment routing |

### 5.2 Claims & Estimation Integrations

| Integration | Direction | Description |
|-------------|-----------|-------------|
| Xactimate / XactAnalysis | Inbound | Property damage estimate import; auto-create payable line items |
| Bill Review Vendor | Bidirectional | Medical bill adjudication and adjustment |
| EDI 835 | Outbound | Remittance advice to medical providers |
| EDI 837 | Inbound | Medical claim submission from providers |
| Litigation Data System | Bidirectional | Litigation case management integration |

### 5.3 Identity & Compliance Integrations

| Integration | Direction | Description |
|-------------|-----------|-------------|
| KYC/Identity Verification | Outbound | Payee identity verification during onboarding |
| Tax ID Verification | Outbound | EIN/SSN validation for reportable payments |

### 5.4 Finance & Document Integrations

| Integration | Direction | Description |
|-------------|-----------|-------------|
| General Ledger | Outbound | Payment and reserve transactions to accounting |
| Accounting System | Bidirectional | Financial reconciliation |
| Document Management System | Bidirectional | Document storage for policies, claims, payments |

### 5.5 Integration Non-Functional Requirements

| Requirement | Description |
|-------------|-------------|
| Error handling | All integrations must implement error handling with specific error codes |
| Retry logic | Exponential backoff retry for transient failures (max 3 retries) |
| Graceful degradation | Integration failures must not block core policy/claim workflows |
| Timeout configuration | Configurable per integration; default 30 seconds |
| Circuit breaker | Circuit breaker pattern for downstream service failures |
| Audit logging | All integration calls (request/response) logged for troubleshooting |

---

## 6. Error Handling Requirements

| Scenario | Display Message |
|----------|----------------|
| No matching policies found | "No matching policies found." |
| No matching claims found | "No matching claims found." |
| System unavailable | "System is currently unavailable." |
| Policy/claim/payment details cannot be retrieved | "Unable to retrieve details. Please try again later." |
| Claim-level policy data cannot be saved | "Unable to save claim-level policy data. Please try again later." |
| No prior claims exist for policy | "No prior claims exist for this policy." |
| Validation error | Field-level error messages with specific constraint violated |
| Payment processing failure | "Payment processing is currently unavailable. Please try again later." |
| Integration service unavailable | Service-specific message + "Please try again later." |

---

## 7. Traceability Matrix

| Requirement | User Story | Business Rule |
|-------------|------------|---------------|
| Policy search (multi-criteria) | US-003 | POL-001, POL-010, POL-012 |
| Policy detail view | US-004 | POL-010 |
| Policy creation | US-005 | POL-001 through POL-007, COV-001 through COV-004, PRM-001 |
| Policy endorsement | US-006 | POL-008, POL-009, PRM-005, COV-006 |
| Policy cancellation | US-007 | POL-004, POL-005, CLM-009 |
| Claim search | US-008 | CLM-008 |
| FNOL submission | US-009 | CLM-001 through CLM-004 |
| Claim detail & history | US-010 | CLM-008 |
| Claim investigation | US-011 | CLM-005, CLM-006, CLM-007 |
| Claim-level policy data | US-012 | CLM-010 |
| Subrogation referral | US-013 | CLM-011 |
| Injury/coding details | US-014 | — |
| Scheduled payments | US-015 | PAY-001, PAY-006 |
| Payment creation | US-016 | PAY-001 through PAY-012 |
| Void/reversal/reissue | US-017 | PAY-004, PAY-010, PAY-011 |
| Vendor/claimant onboarding | US-018 | PAY-003, PAY-009 |
| Estimate import (Xactimate) | US-019 | — |
| EDI/EOB medical payments | US-020 | — |
| Negotiation & settlement | US-021 | CLM-005 |
| Reserve management | US-022 | CLM-012, PAY-006 |
| Audit logging | US-002 | POL-011, NFR-S-006, NFR-S-007 |
| RBAC | US-001 | NFR-S-004 |

---

## 8. Open Questions

| ID | Question | Owner | Priority |
|----|----------|-------|----------|
| OQ-001 | What are the specific rating factors and base rates for each policy type (AUTO, HOME, LIFE, HEALTH, COMMERCIAL)? | Underwriting SME | High |
| OQ-002 | Which specific states require additional regulatory compliance rules beyond the 30-day cancellation notice? | Compliance Team | High |
| OQ-003 | What is the threshold amount for manager approval on settlement overrides and void/reversal actions? | Claims Management | High |
| OQ-004 | Which identity/KYC vendor is to be used for payee onboarding verification? | IT/Procurement | High |
| OQ-005 | What is the specific session idle timeout per role (Agent, Adjuster, Payment Processor)? | Security Team | Medium |
| OQ-006 | Are there state-specific grace periods for premium payment beyond the BRD specification? | Compliance Team | Medium |
| OQ-007 | What is the maximum number of joint payees allowed per payment transaction? | Legal/Finance | Medium |
| OQ-008 | What document types are accepted for attachment to payment transactions and claims? | Business Owner | Medium |
| OQ-009 | Is the General Ledger integration real-time or batch-based (nightly)? | Finance/IT | Medium |
| OQ-010 | Are there jurisdiction-specific rules for 1099 tax reporting thresholds? | Finance/Compliance | Medium |
| OQ-011 | What claim status values map to the UI filter options Open/Closed/Paid/Denied from the internal status enum? | Business Analyst | High |
| OQ-012 | What are the specific EDI 835/837 trading partner requirements (ISA/GS segments, version)? | Integration Team | Medium |
