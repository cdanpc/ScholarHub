# Data Model — ScholarHub Tables & Fields
> Agent: reference this for every GlideRecord query, Business Rule, and widget server script.
> All tables live in scope: `x_snc_scholar`

---

## Table 1: sn_scholar_scholarship

**Purpose:** Scholarship listings created and managed by Providers.
**Auto-number prefix:** `SCH` → e.g., `SCH0001001`

| Field Name | Type | Required | Values / Notes |
|------------|------|----------|----------------|
| `number` | Auto Number | Yes | SCH0001001 format |
| `name` | String (255) | Yes | Official scholarship title |
| `provider` | Reference → sys_user | Yes | Provider's user record |
| `scholarship_type` | Choice | Yes | `merit` / `need_based` / `combined` |
| `gwa_requirement` | Decimal | No | Min GWA e.g. 85.00, 88.00, 90.00, 92.00 |
| `income_cap` | Currency | No | Max annual family income in PHP |
| `course_priority` | String (1000) | No | Comma-separated: "BSIT,BSCS,BSECE" |
| `school_type_preference` | Choice | No | `public` / `private` / `science_hs` / `both` |
| `slots_available` | Integer | Yes | Total slots |
| `slots_remaining` | Integer | No | Auto-decremented on each approval |
| `application_deadline` | Date | Yes | Last day to submit |
| `benefits_description` | HTML | No | Full benefits description |
| `required_documents` | List (string) | Yes | Comma-separated doc types (see Document Types below) |
| `status` | Choice | Yes | `draft` / `open` / `closed` |

**Useful queries:**
```javascript
// Get all open scholarships
var gr = new GlideRecord('sn_scholar_scholarship');
gr.addQuery('status', 'open');
gr.addQuery('application_deadline', '>=', gs.nowDateTime());
gr.orderBy('application_deadline');
gr.query();

// Get scholarships for a specific provider
var gr = new GlideRecord('sn_scholar_scholarship');
gr.addQuery('provider', currentUserId);
gr.query();
```

---

## Table 2: sn_scholar_application

**Purpose:** One record per student per scholarship application.
**Auto-number prefix:** `APP` → e.g., `APP0001001`

| Field Name | Type | Required | Values / Notes |
|------------|------|----------|----------------|
| `number` | Auto Number | Yes | APP0001001 format |
| `scholarship` | Reference → sn_scholar_scholarship | Yes | |
| `applicant` | Reference → sys_user | Yes | Student user record |
| `status` | Choice | Yes | `draft` / `submitted` / `under_review` / `approved` / `denied` |
| `gwa` | Decimal | Yes | Applicant's current GWA |
| `family_income` | Currency | Yes | Annual family income in PHP |
| `course` | String (255) | Yes | Degree program e.g. "BSIT", "BSCS" |
| `year_level` | Choice | Yes | `1st` / `2nd` / `3rd` / `4th` |
| `school_type` | Choice | Yes | `public` / `private` / `science_hs` |
| `honor_rank` | String (255) | No | e.g. "Valedictorian", "With Highest Honors" |
| `submission_date` | DateTime | No | Auto-stamped by BR on submission |
| `reviewer` | Reference → sys_user | No | Assigned provider |
| `review_notes` | String (2000) | No | Provider notes, denial reason |
| `ai_pre_screen_result` | String (4000) | No | JSON string from Claude API |
| `approval_action` | Choice | No | `approve` / `deny` / `request_docs` — set by UI Action or inbound email |

**Status flow:**
```
draft → submitted → under_review → approved
                               → denied
                  → draft (if Request More Docs)
```

**Useful queries:**
```javascript
// Get all applications for a scholarship (provider view)
var gr = new GlideRecord('sn_scholar_application');
gr.addQuery('scholarship', scholarshipSysId);
gr.addQuery('status', '!=', 'draft');
gr.orderByDesc('submission_date');
gr.query();

// Get applicant's own applications
var gr = new GlideRecord('sn_scholar_application');
gr.addQuery('applicant', gs.getUserID());
gr.query();

// Check for duplicate application
var gr = new GlideRecord('sn_scholar_application');
gr.addQuery('applicant', gs.getUserID());
gr.addQuery('scholarship', scholarshipSysId);
gr.addQuery('status', 'NOT IN', 'denied');
gr.query();
return gr.hasNext(); // true = duplicate exists
```

---

## Table 3: sn_scholar_document

**Purpose:** Individual document records linked to an application.

| Field Name | Type | Required | Values / Notes |
|------------|------|----------|----------------|
| `application` | Reference → sn_scholar_application | Yes | Parent application |
| `document_type` | Choice | Yes | See Document Types below |
| `status` | Choice | Yes | `pending` / `submitted` / `verified` / `rejected` |
| `remarks` | String (500) | No | Reviewer remarks |

**Note:** Actual files are stored as ServiceNow Attachments on this record, not in a separate field.

**Document Types (choice values):**
```
psa_birth_cert       PSA Birth Certificate
report_card          Grade 11/12 Report Card
itr                  Income Tax Return (BIR Form 1701)
bir_exemption        BIR Certificate of Tax Exemption
indigency_cert       Certificate of Indigency (from Barangay or DSWD)
good_moral           Certificate of Good Moral Character
id_picture           2x2 ID Picture with white background
barangay_cert        Barangay Certificate of Residency
house_photo          Photo of Residence (front of house)
utility_bill         Proof of utility bills (electricity/water)
honor_cert           Honor Certificate or class ranking certification
recommendation       Recommendation Letter
essay                Personal Essay / Statement of Purpose
medical_cert         Medical Certificate
enrollment_cert      Certificate of Enrollment (for compliance)
```

---

## Table 4: sn_scholar_award

**Purpose:** Approved scholarship grants. Auto-created on application approval.
**Auto-number prefix:** `AWD` → e.g., `AWD0001001`

| Field Name | Type | Required | Values / Notes |
|------------|------|----------|----------------|
| `number` | Auto Number | Yes | AWD0001001 format |
| `application` | Reference → sn_scholar_application | Yes | Source application |
| `scholar` | Reference → sys_user | Yes | The approved applicant |
| `scholarship` | Reference → sn_scholar_scholarship | Yes | The scholarship awarded |
| `coverage_type` | Choice | Yes | `full_tuition` / `partial` / `tuition_and_stipend` |
| `coverage_percentage` | Decimal | No | e.g. 50.00 for 50% tuition discount |
| `stipend_amount` | Currency | No | Monthly stipend in PHP |
| `start_date` | Date | Yes | Award start date |
| `end_date` | Date | Yes | Expected end date |
| `status` | Choice | Yes | `active` / `on_hold` / `terminated` / `completed` |
| `disbursement_status` | Choice | Yes | `pending` / `released` / `on_hold` |

---

## Table 5: sn_scholar_compliance

**Purpose:** Per-semester compliance submissions from active scholars.

| Field Name | Type | Required | Values / Notes |
|------------|------|----------|----------------|
| `award` | Reference → sn_scholar_award | Yes | Parent award |
| `academic_year` | String (20) | Yes | e.g. "2025-2026" |
| `semester` | Choice | Yes | `1st_semester` / `2nd_semester` / `summer` |
| `gwa_this_semester` | Decimal | Yes | GWA earned this semester |
| `units_enrolled` | Integer | Yes | Units enrolled this semester |
| `status` | Choice | Yes | `pending` / `compliant` / `non_compliant` |
| `reviewed_by` | Reference → sys_user | No | Provider reviewer |
| `review_date` | DateTime | No | When compliance was reviewed |
| `remarks` | String (1000) | No | Provider notes |

**Note:** Enrollment cert and grade report are stored as Attachments on this record.

---

## Key Relationships

```
sn_scholar_scholarship (1)
  └── sn_scholar_application (many)  [scholarship field]
        └── sn_scholar_document (many)  [application field]
        └── sn_scholar_award (1)  [application field on award]
              └── sn_scholar_compliance (many)  [award field]
```

---

## Getting sys_id for Reference Fields

```javascript
// When you need the sys_id of a reference field value:
var gr = new GlideRecord('sn_scholar_application');
gr.get('APP0001001'); // by number
var scholarshipSysId = gr.getValue('scholarship'); // returns sys_id string
var scholarshipName = gr.getDisplayValue('scholarship'); // returns display name

// When setting a reference field:
gr.setValue('scholarship', scholarshipSysId); // use sys_id
```

---

## Common Mistakes to Avoid

- **Never use `gr.scholarship` directly** in a Business Rule — use `gr.getValue('scholarship')` to get sys_id or `gr.getDisplayValue('scholarship')` for display value
- **Currency fields store in USD internally** — always display using `getDisplayValue()` to get PHP formatting
- **status field choices are lowercase** — `'submitted'` not `'Submitted'`
- **Auto-number field is read-only** — never try to set `number` field manually
