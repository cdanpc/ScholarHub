# Process Flow — ScholarHub 6-Phase Lifecycle
> Agent: use this to understand what triggers what, who does what, and what each status means.

---

## Phase 1 — Scholarship Setup
**Actor:** Provider (+ Admin publishes if needed)

```
Provider fills Post Scholarship form in portal
    → sn_scholar_scholarship record created (status: draft)
    → Provider reviews and sets status: open
    → Scholarship appears in Student catalog
    → Auto-close BR fires daily: if deadline passed → status: closed
```

**Key fields set:** name, scholarship_type, gwa_requirement, income_cap, course_priority, slots_available, application_deadline, required_documents, status=open

---

## Phase 2 — Discovery & Pre-Screen
**Actor:** Applicant + AI System

```
Applicant browses catalog (only status=open scholarships shown)
    → Applicant clicks Scholarship Detail page
    → Applicant fills eligibility form (gwa, income, course, school_type, honor_rank)
    → Portal calls Integration Hub → Claude API
    → AI returns: { status, matched[], missing[], note }
    → Portal displays eligibility result card (green/yellow/red)
    → Applicant decides to apply (or not)
```

**AI result is advisory only** — Provider makes the final decision. AI result is stored in `ai_pre_screen_result` on the application record.

---

## Phase 3 — Application Submission
**Actor:** Applicant + System (Business Rules)

```
Applicant completes application form
    → Client Scripts: show/hide fields based on scholarship_type
    → Client Scripts: populate required document checklist from scholarship.required_documents
    → Applicant uploads documents (PSA, grades, ITR, etc.)
    → Applicant clicks "Submit Application" UI Action
    → BR sn_scholar_validate_documents fires (Before Insert):
        IF any required doc missing → ABORT, show error list
        IF all docs present → allow insert
    → Application record created (status: draft initially)
    → BR sn_scholar_set_status_on_submit fires (After Insert):
        Sets status = submitted
        Stamps submission_date = now
        Triggers Flow Designer
```

---

## Phase 4 — Review & Decision
**Actor:** Provider + System (Flow Designer)

```
Flow Designer Step 1: Send "New Application Received" email to Provider
Flow Designer Step 2: Set application status = under_review
Flow Designer Step 3: WAIT for one of:
    a) UI Action clicked (Approve / Deny / Request More Docs)
    b) Inbound email reply (APPROVE or DENY [reason])

PATH A — Request More Docs:
    → Provider selects docs needed, enters message
    → "Request for More Documents" email sent to Applicant
    → Application status = draft
    → Flow loops back to wait for resubmission

PATH B — Deny:
    → Provider enters denial reason
    → Application status = denied
    → "Application Denied" email sent to Applicant with reason
    → Flow ends

PATH C — Approve:
    → Application status = approved
    → Flow triggers Award Creation subflow
    → "Application Approved" email sent to Applicant
    → slots_remaining on scholarship decremented by 1
```

---

## Phase 5 — Award & Onboarding
**Actor:** System (automatic on approval)

```
Award Creation subflow fires:
    → New sn_scholar_award record created:
        scholar = application.applicant
        scholarship = application.scholarship
        coverage_type = (from scholarship benefits)
        status = active
        disbursement_status = pending
        start_date = today
    → "Congratulations" email sent to Scholar
    → Scholar can view award in My Awards portal page
    → Scholar gains access to Compliance Submit widget (has active award)
```

---

## Phase 6 — Compliance & Renewal (Repeats Every Semester)
**Actor:** Scholar + Provider + System

```
System: Scheduled trigger fires at semester end
    → "Compliance Submission Due" email sent to Scholar

Scholar: Opens Compliance Submit form in portal
    → Uploads: enrollment certificate, grade report
    → Enters: gwa_this_semester, units_enrolled, academic_year, semester
    → Submits → sn_scholar_compliance record created (status: pending)
    → "New Compliance Submitted" notification sent to Provider

Provider: Opens Compliance Review widget
    → Reviews submitted docs and GWA
    → PATH A (Compliant): Sets status = compliant, award continues
        → "Award Disbursed" email sent after disbursement released
    → PATH B (Non-Compliant): Sets status = non_compliant
        → BR sn_scholar_flag_non_compliant fires:
            Sets award.status = on_hold
            Sends "Non-Compliance Flagged" email to Scholar + Provider

Repeat every semester until award.end_date reached → award.status = completed
```

---

## Status Reference

### sn_scholar_scholarship.status
| Value | Meaning |
|-------|---------|
| `draft` | Provider saving, not yet published |
| `open` | Visible to applicants, accepting applications |
| `closed` | Past deadline or manually closed, no new applications |

### sn_scholar_application.status
| Value | Meaning | Who Can Change |
|-------|---------|----------------|
| `draft` | Not yet submitted, or returned for more docs | Applicant |
| `submitted` | Submitted by applicant, awaiting review | System (BR) |
| `under_review` | Provider is reviewing | System (Flow) |
| `approved` | Provider approved | Provider (UI Action / Email) |
| `denied` | Provider denied | Provider (UI Action / Email) |

### sn_scholar_award.status
| Value | Meaning |
|-------|---------|
| `active` | Award is current and disbursements can be released |
| `on_hold` | Non-compliance detected, disbursement paused |
| `terminated` | Award cancelled before end date |
| `completed` | Award period ended |

### sn_scholar_compliance.status
| Value | Meaning |
|-------|---------|
| `pending` | Submitted by scholar, awaiting provider review |
| `compliant` | Provider confirmed compliance |
| `non_compliant` | Provider flagged non-compliance |

---

## What Triggers the Flow Designer

The Flow Designer `ScholarHub Application Lifecycle` is triggered by:
```
Table: sn_scholar_application
Trigger type: Record Created
Condition: status = submitted
```

The flow does NOT fire on:
- Draft records
- Records updated to submitted (only newly submitted)
- Compliance records (different flow if built)

---

## The Inbound Email Approval Path

When a provider receives the "New Application Received" email, the reply-to address is configured for inbound processing. Provider can reply with:

```
APPROVE
```
or
```
DENY [reason text here]
```

The Inbound Email Action script:
1. Reads `email.body_text` for the keyword
2. Extracts the application sys_id from the email subject (included in the template)
3. Sets `approval_action` field on the application record
4. The Flow Designer wait condition checks this field and routes to the correct branch
