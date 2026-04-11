# Testing Checklist — ScholarHub
> Agent: run these test cases before marking any phase complete. Use impersonation for every test.

---

## How to Impersonate Users

```
1. Click your user avatar (top right of ServiceNow)
2. Click "Impersonate User"
3. Search for the test user (e.g., applicant@scholarhub.test)
4. Click "Impersonate"
5. Navigate to the portal: /scholarhub
6. Test as that user
7. To stop: click avatar → "End Impersonation"

Always test with the correct user — never assume admin behavior matches user behavior.
```

---

## How to Check Errors

```
System Log (all errors):
  Navigate to: /syslog_list.do?sysparm_query=level=1^ORlevel=2

Filter to ScholarHub errors only:
  Add filter: Source CONTAINS ScholarHub

Flow Designer execution log:
  Flow Designer → your flow → Execution Details tab

Integration Hub log:
  /sn_ih_log_list.do → filter by REST Message = claude_messages

Email sent log:
  /sys_email_log_list.do → filter recent

Browser JavaScript errors:
  F12 → Console tab → look for red errors during portal interactions
```

---

## 12 Required Test Cases (Phase 6 — Run on Day 6)

### TC01: Provider Posts Scholarship
```
User: provider@scholarhub.test
Steps:
  1. Log into portal → navigate to My Scholarships
  2. Click "Post New Scholarship"
  3. Fill all fields (type: need-based, GWA: 85, income: 300000, course: BSIT, slots: 5)
  4. Click Submit / Save
Expected:
  - New scholarship appears in My Scholarships list
  - Status = open
  - Scholarship visible in Student catalog
Pass/Fail: [ ]
```

### TC02: AI Pre-screen — Eligible Result
```
User: applicant@scholarhub.test
Steps:
  1. Log into portal → browse catalog
  2. Click on a scholarship
  3. Fill eligibility form: GWA=90, income=200000, course=BSIT, school=public
  4. Click "Check My Eligibility"
Expected:
  - Green result card appears within 10 seconds
  - "You appear eligible" message
  - Matched criteria list shows GWA and income
  - Missing list is empty
Pass/Fail: [ ]
```

### TC03: AI Pre-screen — Partial / Not Eligible
```
User: applicant@scholarhub.test
Steps:
  1. Same scholarship as TC02
  2. Fill eligibility form: GWA=70, income=600000, course=BSN
  3. Click "Check My Eligibility"
Expected:
  - Red or yellow result card
  - GWA below minimum shown in missing[] list
  - Income over cap shown in missing[] list
Pass/Fail: [ ]
```

### TC04: Application Submission — Valid
```
User: applicant@scholarhub.test
Steps:
  1. On scholarship detail → click "Apply Now"
  2. Fill application: GWA=90, income=200000, course=BSIT, year_level=2nd, school_type=public
  3. Upload mock documents for each required type
  4. Click "Submit Application"
Expected:
  - Application record created with status = submitted
  - Provider receives "New Application Received" email
  - Applicant sees application in My Applications with status = submitted
Pass/Fail: [ ]
```

### TC05: Business Rule Blocks Incomplete Submission
```
User: applicant@scholarhub.test
Steps:
  1. Start a new application
  2. Fill profile fields but upload ONLY 1 of 3 required documents
  3. Click "Submit Application"
Expected:
  - Submission is BLOCKED (setAbortAction)
  - Error message lists the missing document types
  - No application record created in the database
Pass/Fail: [ ]
```

### TC06: Provider Approves Application
```
User: provider@scholarhub.test (after TC04 created an application)
Steps:
  1. Log into portal → Application Queue
  2. Find the TC04 application (status: under_review)
  3. Click the application → review detail page
  4. Click "Approve"
Expected:
  - Application status changes to approved
  - Award record automatically created
  - Applicant receives "Application Approved" congratulations email
  - Slots remaining on scholarship decremented by 1
Pass/Fail: [ ]
```

### TC07: Provider Denies Application with Reason
```
User: provider@scholarhub.test
Steps:
  1. Find a submitted application in the queue
  2. Click "Deny"
  3. Enter denial reason when prompted: "GWA does not meet our minimum requirement"
  4. Confirm
Expected:
  - Application status = denied
  - review_notes = "GWA does not meet our minimum requirement"
  - Applicant receives "Application Denied" email including the reason
Pass/Fail: [ ]
```

### TC08: Request More Documents → Resubmission
```
User: provider@scholarhub.test
Steps:
  1. Find a submitted application
  2. Click "Request More Documents"
  3. Enter: "Please upload your ITR or Certificate of Indigency"
  4. Confirm
Expected:
  - Application status = draft
  - review_notes contains the provider's message
  - Applicant receives email with the message
  
Switch to applicant@scholarhub.test:
  5. Log in → My Applications → find the draft application
  6. Upload the missing document
  7. Resubmit
Expected:
  - Application status = submitted again
  - Provider receives new notification
Pass/Fail: [ ]
```

### TC09: Email Inbound Approval
```
Pre-req: Have a real email inbox for the provider test account
Steps:
  1. Submit an application as applicant@scholarhub.test
  2. Check provider's email inbox for "New Application Received"
  3. Reply to that email with exactly: APPROVE
Expected:
  - ServiceNow processes the reply
  - Application status = approved
  - Award record created
  - Applicant receives approval email
Pass/Fail: [ ]
```

### TC10: Compliance Submission and Review
```
Pre-req: TC06 completed (award exists for applicant)
User: applicant@scholarhub.test
Steps:
  1. Log into portal → navigate to Compliance page
  2. Fill: academic_year=2025-2026, semester=1st_semester, gwa=88, units=21
  3. Upload mock enrollment cert and grade report
  4. Submit
Expected:
  - Compliance record created with status = pending
  - Provider receives notification

Switch to provider@scholarhub.test:
  5. Navigate to Scholar Roster → find the scholar
  6. Click "View Compliance" → find the pending compliance
  7. Click "Mark Compliant"
Expected:
  - Compliance status = compliant
  - Award remains active
Pass/Fail: [ ]
```

### TC11: Non-Compliance Flags Award
```
Pre-req: TC06 completed
User: provider@scholarhub.test
Steps:
  1. Open a compliance record for a scholar
  2. Set status = non_compliant
  3. Save
Expected:
  - Business Rule fires: sn_scholar_flag_non_compliant
  - Award status = on_hold
  - Award disbursement_status = on_hold
  - Scholar receives "Non-Compliance Flagged" email
Pass/Fail: [ ]
```

### TC12: Role Isolation Verification
```
Steps:
  1. Impersonate applicant@scholarhub.test
  2. Attempt to navigate to provider pages (/scholarhub?id=applications)
  Expected: Provider widgets are hidden or access is denied
  
  3. Impersonate provider@scholarhub.test
  4. Attempt to submit a student application
  Expected: "Submit Application" UI Action is not visible (condition restricts to applicant role)
  
  5. Verify applicant cannot see another applicant's applications
  (create a second applicant user and verify their applications are not visible)
Pass/Fail: [ ]
```

---

## Quick Sanity Checks (Run These First)

Before running full test cases, verify the basics:

```
[ ] ServiceNow instance is accessible (not hibernated)
[ ] All 5 tables exist and have correct fields
[ ] All 3 test users exist with correct roles
[ ] 3 sample scholarships are seeded (status=open)
[ ] Service Portal is accessible at /scholarhub
[ ] Claude API connection test passes in Integration Hub
[ ] At least one email notification sends successfully
[ ] Studio shows "Up to date" with GitHub
```

---

## Debugging Tips

```
Application not submitting:
  → Check sys_log for BR errors
  → Verify BR is Active and on correct table/trigger
  → Test with background script: does the BR logic work standalone?

Widget not showing data:
  → Check browser console for JS errors
  → Check widget server script with gs.log() statements
  → Refresh with ?sysparm_no_cache=true

Flow not triggering:
  → Check Flow Designer trigger condition (status = 'submitted' exact value)
  → Check Flow execution log for errors
  → Verify the application record has status=submitted (not 'Submitted')

Email not received:
  → Check /sys_email_log_list.do for sent emails
  → Verify notification is Active
  → Verify recipient email address is correct
  → Check spam folder of test email

AI pre-screen returns error:
  → Check Integration Hub execution log
  → Verify API key is correct in System Property or Credential
  → Test REST Method manually in Integration Hub
  → Check if Claude API is responding at: https://api.anthropic.com/v1/messages
```
