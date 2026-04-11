# Notification Guide — ServiceNow Email Setup
> Agent: ScholarHub has 9 outbound templates and 1 inbound action. All are required for the rubric.

---

## Outbound Notification Setup

Navigate to: **Studio → Email → Notifications → New**

```
Required fields:
  Name:          [descriptive name e.g., "ScholarHub - Application Approved"]
  Table:         sn_scholar_application (or the relevant table)
  Send when:     [choose: Record inserted / Record updated / Event fired]
  Condition:     [when this notification fires]
  Recipients:    [email, user reference, or script]
  Subject:       [subject line with ${tokens}]
  Message HTML:  [HTML email body with ${tokens}]
  Active:        YES
```

---

## Token Reference (${field_name} syntax)

```
On sn_scholar_application:
  ${number}                  → APP0001001
  ${applicant.name}          → Juan dela Cruz
  ${applicant.email}         → juan@email.com
  ${scholarship.name}        → SM Foundation Scholarship
  ${scholarship.provider.name} → SM Foundation
  ${status}                  → Submitted (display value)
  ${submission_date}         → 05/12/2026
  ${review_notes}            → Denial reason or provider notes
  ${gwa}                     → 90.5

On sn_scholar_award:
  ${scholar.name}            → Juan dela Cruz
  ${scholarship.name}        → SM Foundation Scholarship
  ${coverage_type}           → Full Tuition
  ${stipend_amount}          → ₱7,000.00
  ${start_date}              → 05/12/2026
```

---

## All 9 Notification Templates

### T1: New Application Received
```
Table:     sn_scholar_application
When:      Record inserted AND status = submitted
To:        ${scholarship.provider.email}
Subject:   [ScholarHub] New Application: ${number} - ${applicant.name}
Body:
  Dear ${scholarship.provider.name},

  A new scholarship application has been received.

  Application: ${number}
  Applicant: ${applicant.name}
  Scholarship: ${scholarship.name}
  Submitted: ${submission_date}

  Please log in to review this application:
  [PORTAL LINK]

  To APPROVE, reply to this email with: APPROVE
  To DENY, reply with: DENY [reason]

  Regards,
  ScholarHub System
```

### T2: Missing Documents Alert
```
Table:     sn_scholar_application
When:      Flow Designer sends this / status changes to draft (with notes)
To:        ${applicant.email}
Subject:   [ScholarHub] Action Required: Missing Documents - ${number}
Body:
  Dear ${applicant.name},

  Your application ${number} for ${scholarship.name} requires additional documents.

  Message from the provider:
  ${review_notes}

  Please log in and upload the missing documents:
  [PORTAL LINK]

  Deadline: ${scholarship.application_deadline}
```

### T3: Application Under Review
```
Table:     sn_scholar_application
When:      status changes to under_review
To:        ${applicant.email}
Subject:   [ScholarHub] Your Application is Under Review - ${number}
Body:
  Dear ${applicant.name},

  Your application ${number} for ${scholarship.name} is now under review.
  We will notify you once a decision has been made.
```

### T4: Request for More Documents
```
(Same trigger as T2 — can combine into one notification or keep separate)
Subject:   [ScholarHub] Please Resubmit Documents - ${number}
```

### T5: Application Approved
```
Table:     sn_scholar_application
When:      status changes to approved
To:        ${applicant.email}
Subject:   [ScholarHub] Congratulations! Application Approved - ${number}
Body:
  Dear ${applicant.name},

  Congratulations! Your application for ${scholarship.name} has been APPROVED.

  Your scholarship award details are now available in your portal.
  Log in to view your award: [PORTAL LINK]

  Welcome to the ScholarHub scholar community!
```

### T6: Application Denied
```
Table:     sn_scholar_application
When:      status changes to denied
To:        ${applicant.email}
Subject:   [ScholarHub] Application Update - ${number}
Body:
  Dear ${applicant.name},

  We regret to inform you that your application ${number} for
  ${scholarship.name} was not approved.

  Reason: ${review_notes}

  You are encouraged to apply for other available scholarships.
  [CATALOG LINK]
```

### T7: Award Disbursed
```
Table:     sn_scholar_award
When:      disbursement_status changes to released
To:        ${scholar.email}
Subject:   [ScholarHub] Scholarship Disbursement Released
Body:
  Dear ${scholar.name},

  Your scholarship disbursement for ${scholarship.name} has been released.

  Coverage: ${coverage_type}
  Amount/Stipend: ${stipend_amount}

  Please coordinate with your school registrar for processing.
```

### T8: Compliance Submission Due
```
Table:     sn_scholar_compliance (or via scheduled flow)
When:      Scheduled / Flow Designer trigger
To:        ${award.scholar.email}
Subject:   [ScholarHub] Compliance Documents Due
Body:
  Dear ${award.scholar.name},

  Your compliance documents for ${award.scholarship.name} are due.

  Period: ${semester} ${academic_year}

  Please submit: Grade report + Certificate of Enrollment
  [COMPLIANCE PORTAL LINK]
```

### T9: Non-Compliance Flagged
```
Table:     sn_scholar_compliance
When:      status changes to non_compliant
To:        ${award.scholar.email}, ${award.scholarship.provider.email}
Subject:   [ScholarHub] Compliance Issue - Action Required
Body:
  Your scholarship award has been placed ON HOLD due to a compliance issue.
  ${remarks}
  Please contact your scholarship provider to resolve this issue.
```

---

## Inbound Email Action

Navigate to: **All Applications → System Policy → Email → Inbound Actions → New**

```
Name:         ScholarHub Approve/Deny via Email Reply
Active:        YES
Order:         100
Type:          Record Action
Table:         sn_scholar_application
Conditions:    email.subject CONTAINS 'ScholarHub'
               AND email.subject CONTAINS 'APP'

Script:
```
```javascript
// Parse approval command from email body
var body = email.body_text.trim().toUpperCase();
var subject = email.subject;

// Extract application number from subject
var appNumMatch = subject.match(/APP\d+/);
if (!appNumMatch) return;
var appNumber = appNumMatch[0];

// Find the application record
var gr = new GlideRecord('sn_scholar_application');
gr.addQuery('number', appNumber);
gr.addQuery('status', 'under_review');
gr.setLimit(1);
gr.query();

if (!gr.next()) {
    gs.log('ScholarHub Inbound: Application ' + appNumber + ' not found or not under review', 'ScholarHub');
    return;
}

// Parse the command
if (body.startsWith('APPROVE')) {
    gr.setValue('approval_action', 'approve');
    gr.update();
    gs.log('ScholarHub Inbound: Approved ' + appNumber, 'ScholarHub');
} else if (body.startsWith('DENY')) {
    var reason = email.body_text.substring(4).trim(); // text after DENY
    if (!reason) reason = 'Denied by provider via email.';
    gr.setValue('approval_action', 'deny');
    gr.setValue('review_notes', reason);
    gr.update();
    gs.log('ScholarHub Inbound: Denied ' + appNumber + ' - ' + reason, 'ScholarHub');
}
```

---

## Testing Notifications

```
1. Set all notification recipients to a test email address (e.g., team Gmail)
2. Trigger the event (submit application, approve, etc.)
3. Check the test inbox within 60 seconds
4. If not received: check System Log for email errors
5. Also check: Outbound Email Activity (navigate to /sys_email_log_list.do)
```

**Common issues:**
- Email not sent: check the notification is Active and condition matches
- Tokens show as `${field_name}` literally: the field name is wrong — check data-model.md for exact field names
- Reply-to for inbound: must match exactly what ServiceNow is monitoring
