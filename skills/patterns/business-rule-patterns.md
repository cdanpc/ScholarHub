# Business Rule Patterns — ServiceNow
> Agent: every BR in ScholarHub follows these exact patterns. No variations.

---

## Business Rule Structure

```javascript
// Business Rule configuration:
// Name: sn_scholar_[action_description]
// Table: sn_scholar_[table_name]
// When: Before or After
// Insert/Update/Delete/Query: check as appropriate
// Active: true
// Order: 100 (default, adjust if sequencing matters)

// Script body:
(function executeRule(current, previous /*null when inserting*/) {
    // Your logic here
    // Use current.getValue(), current.setValue()
    // Use gs.addErrorMessage() for user-facing errors
    // Use gs.log() for debugging (check sys_log)
})(current, previous);
```

---

## When to Use Which Trigger

| Trigger | Use When |
|---------|---------|
| **Before Insert** | Validate data before record is saved; abort if invalid; modify field values before save |
| **After Insert** | Trigger workflows; create related records; send notifications; decrement counters |
| **Before Update** | Validate changes; check field transitions; abort invalid updates |
| **After Update** | React to status changes; update related records |
| **Async** | Long-running operations; things that don't need to complete before the user sees a response |
| **Scheduled** | Periodic jobs (daily auto-close, compliance reminders) |

---

## ScholarHub Business Rules

### BR 1: sn_scholar_validate_documents

```javascript
// Table: sn_scholar_application
// When: Before Insert
// Condition: status == 'submitted' (check "Condition" field: current.status == 'submitted')

(function executeRule(current, previous) {
    var scholarshipSysId = current.getValue('scholarship');
    var applicationSysId = current.getUniqueValue();

    // Get required documents from scholarship
    var schGR = new GlideRecord('sn_scholar_scholarship');
    if (!schGR.get(scholarshipSysId)) {
        gs.addErrorMessage('Scholarship not found.');
        current.setAbortAction(true);
        return;
    }

    var requiredDocs = schGR.getValue('required_documents');
    if (!requiredDocs) return; // No requirements defined — allow

    var required = requiredDocs.split(',');
    var missing = [];

    // Check each required document type
    for (var i = 0; i < required.length; i++) {
        var docType = required[i].trim();
        var docGR = new GlideRecord('sn_scholar_document');
        docGR.addQuery('application', applicationSysId);
        docGR.addQuery('document_type', docType);
        docGR.addQuery('status', '!=', 'pending');
        docGR.setLimit(1);
        docGR.query();
        if (!docGR.hasNext()) {
            missing.push(docType.replace(/_/g, ' ').toUpperCase());
        }
    }

    if (missing.length > 0) {
        gs.addErrorMessage('Missing required documents: ' + missing.join(', ') +
            '. Please upload all required documents before submitting.');
        current.setAbortAction(true);
    }
})(current, previous);
```

### BR 2: sn_scholar_set_status_on_submit

```javascript
// Table: sn_scholar_application
// When: After Insert
// Condition: leave blank (runs on all inserts)

(function executeRule(current, previous) {
    if (current.getValue('status') != 'submitted') return;

    // Stamp submission date
    var gr = new GlideRecord('sn_scholar_application');
    if (gr.get(current.getUniqueValue())) {
        gr.setValue('submission_date', gs.nowDateTime());
        gr.update();
    }
    // Note: Flow Designer is triggered separately via configured trigger,
    // not from this BR. This BR just handles the timestamp.
})(current, previous);
```

### BR 3: sn_scholar_prevent_duplicate

```javascript
// Table: sn_scholar_application
// When: Before Insert
// Condition: current.isNewRecord()

(function executeRule(current, previous) {
    var applicantId = current.getValue('applicant');
    var scholarshipId = current.getValue('scholarship');

    var existing = new GlideRecord('sn_scholar_application');
    existing.addQuery('applicant', applicantId);
    existing.addQuery('scholarship', scholarshipId);
    existing.addQuery('status', 'NOT IN', 'denied');
    existing.setLimit(1);
    existing.query();

    if (existing.hasNext()) {
        gs.addErrorMessage('You already have an active application for this scholarship.');
        current.setAbortAction(true);
    }
})(current, previous);
```

### BR 4: sn_scholar_flag_non_compliant

```javascript
// Table: sn_scholar_compliance
// When: Before Update
// Condition: current.status.changes() && current.status == 'non_compliant'

(function executeRule(current, previous) {
    if (!current.status.changes()) return;
    if (current.getValue('status') != 'non_compliant') return;

    var awardSysId = current.getValue('award');
    var awardGR = new GlideRecord('sn_scholar_award');
    if (awardGR.get(awardSysId)) {
        awardGR.setValue('status', 'on_hold');
        awardGR.setValue('disbursement_status', 'on_hold');
        awardGR.update();
        gs.log('ScholarHub: Award ' + awardGR.getValue('number') +
               ' placed on hold due to non-compliance.', 'ScholarHub');
    }
})(current, previous);
```

### BR 5: sn_scholar_auto_close_scholarships (Scheduled)

```javascript
// Type: Scheduled Script Execution
// Run: Daily
// Script:
var gr = new GlideRecord('sn_scholar_scholarship');
gr.addQuery('status', 'open');
gr.addQuery('application_deadline', '<', gs.nowDate()); // deadline is today or past
gr.query();
var count = 0;
while (gr.next()) {
    gr.setValue('status', 'closed');
    gr.update();
    count++;
}
if (count > 0) {
    gs.log('ScholarHub: Auto-closed ' + count + ' expired scholarships.', 'ScholarHub');
}
```

---

## Common BR Debugging

```javascript
// Log to System Log (check at: /syslog_list.do)
gs.log('ScholarHub: variable value = ' + someVar, 'ScholarHub');

// Add info message (shows in green banner to user)
gs.addInfoMessage('Application saved successfully.');

// Add error message (shows in red banner, does NOT abort automatically)
gs.addErrorMessage('Please fix: ' + errorDetail);

// Abort the action AND show error
gs.addErrorMessage('Cannot submit: ' + errorDetail);
current.setAbortAction(true);
return; // always return after aborting

// Check if this is a new record
current.isNewRecord();          // true for inserts
current.operation();            // 'insert', 'update', 'delete'
```

---

## Business Rule Conditions Field

Use the Conditions field (not script) for simple checks — it's faster:
```
// Simple condition examples (in the Condition field):
current.status == 'submitted'
current.status.changes()
current.status.changesTo('non_compliant')
current.isNewRecord()
```

Use a script condition only when the logic is complex.
