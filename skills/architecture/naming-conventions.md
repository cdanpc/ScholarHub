# Naming Conventions — ScholarHub
> Agent: use these names exactly. Consistent naming prevents merge conflicts and confusion across 5 members.

---

## Table Names

```
sn_scholar_scholarship     Scholarship listings
sn_scholar_application     Student applications
sn_scholar_document        Application documents
sn_scholar_award           Approved awards
sn_scholar_compliance      Compliance submissions
```

Rule: All tables use `sn_scholar_` prefix. Never deviate.

---

## Business Rules

```
Format: sn_scholar_[action]_[trigger]
Examples:
  sn_scholar_validate_documents
  sn_scholar_set_status_on_submit
  sn_scholar_prevent_duplicate
  sn_scholar_flag_non_compliant
  sn_scholar_auto_close_scholarships
```

---

## Client Scripts

```
Format: [Descriptive action] on [table short name]
Examples:
  Show Income Fields for Need-Based   (on sn_scholar_application)
  Dynamic Document Checklist          (on sn_scholar_application)
  GWA Eligibility Indicator          (on sn_scholar_application)
  Lock Form After Decision           (on sn_scholar_application)
```

---

## UI Actions

```
Format: [Verb] [Object]
Examples:
  Submit Application
  Approve Application
  Deny Application
  Request More Documents
  Release Disbursement
```

---

## Script Includes

```
Format: Scholar[FeatureName]  (PascalCase)
Examples:
  ScholarUtils              General utility functions
  AIEligibilityHelper       AI prompt building and response parsing
```

---

## Service Portal

```
Portal ID:      scholarhub
Page IDs:       homepage, scholarship, apply, my-apps, my-awards,
                compliance, my-schemas, applications, review, scholars

Widget IDs:     Format: scholar-[description] (kebab-case)
Examples:
  scholar-catalog
  scholar-detail
  scholar-apply
  scholar-my-apps
  scholar-my-awards
  scholar-compliance-submit
  scholar-my-scholarships
  scholar-post-scholarship
  scholar-review-queue
  scholar-review-detail
  scholar-roster
  scholar-ai-check
```

---

## Email Notifications

```
Format: ScholarHub - [Event Description]
Examples:
  ScholarHub - New Application Received
  ScholarHub - Missing Documents Alert
  ScholarHub - Application Under Review
  ScholarHub - Request for More Documents
  ScholarHub - Application Approved
  ScholarHub - Application Denied
  ScholarHub - Award Disbursed
  ScholarHub - Compliance Submission Due
  ScholarHub - Non-Compliance Flagged
```

---

## Flow Designer

```
Flows:
  ScholarHub Application Lifecycle    (main flow)

Subflows:
  Create Award Record                 (called on approval)
  Trigger Compliance Reminder         (scheduled)

Flow Variables (camelCase):
  applicationSysId
  scholarshipSysId
  applicantSysId
  applicantEmail
  providerSysId
  providerEmail
  approvalAction
  awardSysId
```

---

## System Properties

```
Format: x_snc_scholar.[property_name]
Examples:
  x_snc_scholar.claude_api_key        Claude API key (encrypted)
  x_snc_scholar.portal_url            ScholarHub portal base URL
  x_snc_scholar.admin_email           Admin notification email
```

---

## GitHub

```
Repository:     scholarhub-servicenow
Branches:
  main                    stable, demo-ready
  feature/m1-backend      Member 1 work
  feature/m2-workflow     Member 2 work
  feature/m3-portal       Member 3 work
  feature/m4-ai           Member 4 work
  feature/m5-ui-docs      Member 5 work
  fix/[bug-description]   Bug fixes

Commit messages:
  feat: [description]     New feature
  fix: [description]      Bug fix
  docs: [description]     Documentation
  test: [description]     Test scenarios
  refactor: [description] Code improvement

Tag:
  v1.0-hackathon          Final hackathon submission tag
```

---

## Test Users

```
applicant@scholarhub.test     Student test account
provider@scholarhub.test      Provider test account
admin@scholarhub.test         Admin test account
```

---

## Status Choice Values (Always Lowercase)

```
Scholarship status:    draft, open, closed
Application status:    draft, submitted, under_review, approved, denied
Award status:          active, on_hold, terminated, completed
Disbursement status:   pending, released, on_hold
Compliance status:     pending, compliant, non_compliant
Document status:       pending, submitted, verified, rejected

Scholarship type:      merit, need_based, combined
Year level:            1st, 2nd, 3rd, 4th
School type:           public, private, science_hs
Semester:              1st_semester, 2nd_semester, summer
Coverage type:         full_tuition, partial, tuition_and_stipend
```
