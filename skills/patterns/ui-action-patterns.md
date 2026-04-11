# UI Action Patterns — ServiceNow
> Agent: UI Actions are buttons on forms and list views. These are the action buttons judges will click in the demo.

---

## UI Action Configuration Fields

| Field | Description |
|-------|-------------|
| Name | Display label on the button |
| Table | Which table's form/list it appears on |
| Active | Must be true |
| Client | Check if script runs client-side first (for prompts/confirmations) |
| Onclick | Client-side JavaScript function name (if Client is checked) |
| Condition | When the button appears (leave blank = always, or write JavaScript condition) |
| Script | Server-side script (runs after client completes) |
| Action name | Internal identifier |
| Form button | Show on form (check this) |
| List button | Show on list (usually unchecked for ScholarHub) |
| Roles | Restrict to specific roles (leave blank = all, or add sn_scholar.provider etc.) |

---

## ScholarHub UI Actions

### UA 1: Submit Application

```
Name: Submit Application
Table: sn_scholar_application
Condition: current.status == 'draft' && gs.hasRole('sn_scholar.applicant')
Client: false
Roles: sn_scholar.applicant

Script:
```
```javascript
// Server-side script
var applicationId = current.getUniqueValue();
current.setValue('status', 'submitted');
current.update();
// Flow Designer trigger fires automatically on status change to submitted
gs.addInfoMessage('Your application has been submitted successfully.');
action.setRedirectURL(current);
```

### UA 2: Approve Application

```
Name: Approve
Table: sn_scholar_application
Condition: current.status == 'under_review' && gs.hasRole('sn_scholar.provider')
Client: false
Roles: sn_scholar.provider

Script:
```
```javascript
current.setValue('status', 'approved');
current.setValue('approval_action', 'approve');
current.update();

// Create award record
var awardGR = new GlideRecord('sn_scholar_award');
awardGR.initialize();
awardGR.setValue('application', current.getUniqueValue());
awardGR.setValue('scholar', current.getValue('applicant'));
awardGR.setValue('scholarship', current.getValue('scholarship'));
awardGR.setValue('coverage_type', 'full_tuition');
awardGR.setValue('status', 'active');
awardGR.setValue('disbursement_status', 'pending');
awardGR.setValue('start_date', gs.nowDate());
awardGR.insert();

// Decrement slots
var schGR = new GlideRecord('sn_scholar_scholarship');
if (schGR.get(current.getValue('scholarship'))) {
    var remaining = parseInt(schGR.getValue('slots_remaining') || schGR.getValue('slots_available')) - 1;
    schGR.setValue('slots_remaining', Math.max(0, remaining));
    schGR.update();
}

gs.addInfoMessage('Application approved. Award record created.');
action.setRedirectURL(current);
```

### UA 3: Deny Application (with client-side prompt)

```
Name: Deny
Table: sn_scholar_application
Condition: current.status == 'under_review' && gs.hasRole('sn_scholar.provider')
Client: true
Onclick: denyApplication()
Roles: sn_scholar.provider

Client script (Onclick field):
```
```javascript
function denyApplication() {
    var reason = prompt('Please enter the reason for denial:');
    if (!reason || reason.trim() == '') {
        alert('A denial reason is required.');
        return false;
    }
    g_form.setValue('review_notes', reason);
    gsftSubmit(null, g_form.getFormElement(), 'deny_application');
}
```
```javascript
// Server-side Script:
current.setValue('status', 'denied');
current.setValue('approval_action', 'deny');
// review_notes was set by client script before submit
current.update();
gs.addInfoMessage('Application has been denied.');
action.setRedirectURL(current);
```

### UA 4: Request More Documents

```
Name: Request More Documents
Table: sn_scholar_application
Condition: current.status == 'under_review' && gs.hasRole('sn_scholar.provider')
Client: true
Onclick: requestMoreDocs()

Client script:
```
```javascript
function requestMoreDocs() {
    var message = prompt('Specify which documents are needed:');
    if (!message || message.trim() == '') {
        alert('Please specify which documents are missing.');
        return false;
    }
    g_form.setValue('review_notes', 'Additional documents needed: ' + message);
    gsftSubmit(null, g_form.getFormElement(), 'request_docs');
}
```
```javascript
// Server script:
current.setValue('status', 'draft');
current.setValue('approval_action', 'request_docs');
current.update();
gs.addInfoMessage('Applicant has been notified to resubmit documents.');
action.setRedirectURL(current);
```

### UA 5: Release Disbursement

```
Name: Release Disbursement
Table: sn_scholar_award
Condition: current.status == 'active' && current.disbursement_status == 'pending' && gs.hasRole('sn_scholar.provider')
Client: false
Roles: sn_scholar.provider

Script:
```
```javascript
current.setValue('disbursement_status', 'released');
current.update();
gs.addInfoMessage('Disbursement released. Scholar will be notified.');
action.setRedirectURL(current);
```

---

## Common UI Action Mistakes

- **Condition must be JavaScript** — `current.status == 'under_review'` not `status=under_review`
- **Client prompt then server script**: use `gsftSubmit()` in client to trigger server side
- **`action.setRedirectURL(current)`** — always redirect after update or the form shows stale data
- **Roles field**: leave blank = available to all; fill in role = restricted. Don't rely on Condition alone for security — also set the Roles field.
