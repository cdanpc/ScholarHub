# Validation Patterns — ScholarHub
> Agent: validation in ScholarHub happens at THREE layers. Never rely on just one.
> Client validates for UX. Server validates for integrity. Flow validates for business rules.

---

## The Three-Layer Validation Strategy

```
LAYER 1 — Client (Browser)
  When: As the user types or tabs between fields
  How:  Client Scripts using g_form API + GlideAjax for server data
  Goal: Immediate feedback, prevent bad form state before submit
  Catches: Empty required fields, format errors, range violations, GWA below threshold

LAYER 2 — Server (Business Rule)
  When: Before the record is inserted or updated in the database
  How:  Business Rule (Before Insert/Update) with setAbortAction
  Goal: Data integrity guarantee — no bad record ever saved
  Catches: Missing documents, duplicate applications, expired deadlines, invalid transitions

LAYER 3 — Flow / Integration
  When: After record is saved, during workflow processing
  How:  Flow Designer conditions, Script Actions, error branches
  Goal: Business rule enforcement — provider decisions, compliance checks
  Catches: Non-compliant scholars, approval state mismatches, API failures
```

**Rule: Never trust only the client. Never skip the server. The server layer is the ground truth.**

---

## Layer 1 — Client-Side Validation

### Real-Time Field Validation (onChange)

```javascript
// Client Script: GWA Range Check
// Table: sn_scholar_application | Type: onChange | Field: gwa

function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading || newValue === '') return;

    var gwa = parseFloat(newValue);

    // Basic range validation — no server call needed
    if (isNaN(gwa) || gwa < 0 || gwa > 100) {
        g_form.showFieldMsg('gwa', 'GWA must be a number between 0 and 100.', 'error');
        return;
    }

    // Clear any previous error
    g_form.hideFieldMsg('gwa');

    // Compare against scholarship requirement (requires server call)
    var scholarshipId = g_form.getValue('scholarship');
    if (!scholarshipId) return; // No scholarship selected yet

    var ga = new GlideAjax('ScholarUtils');
    ga.addParam('sysparm_name', 'getGWARequirement');
    ga.addParam('sysparm_scholarship_id', scholarshipId);
    ga.getXML(function(response) {
        var required = parseFloat(response.responseXML.documentElement.getAttribute('answer'));
        if (isNaN(required)) return;

        if (gwa < required) {
            g_form.showFieldMsg('gwa',
                'GWA ' + gwa + ' is below the minimum requirement of ' + required + '. You may still apply.',
                'warning');
        } else {
            g_form.showFieldMsg('gwa',
                'GWA ' + gwa + ' meets the minimum requirement of ' + required + '.',
                'info');
        }
    });
}
```

### Multi-Field Dependency Validation (onSubmit)

```javascript
// Client Script: Pre-Submit Validation
// Table: sn_scholar_application | Type: onSubmit

function onSubmit() {
    var isValid = true;
    var errors = [];

    // Required field checks
    var requiredFields = [
        { field: 'gwa',           label: 'GWA' },
        { field: 'family_income', label: 'Annual family income' },
        { field: 'course',        label: 'Course / program' },
        { field: 'year_level',    label: 'Year level' },
        { field: 'school_type',   label: 'School type' }
    ];

    for (var i = 0; i < requiredFields.length; i++) {
        var val = g_form.getValue(requiredFields[i].field);
        if (!val || val.trim() === '') {
            errors.push(requiredFields[i].label + ' is required.');
            g_form.showFieldMsg(requiredFields[i].field, 'This field is required.', 'error');
            isValid = false;
        }
    }

    // GWA range check
    var gwa = parseFloat(g_form.getValue('gwa'));
    if (!isNaN(gwa) && (gwa < 0 || gwa > 100)) {
        errors.push('GWA must be between 0 and 100.');
        isValid = false;
    }

    // Income must be positive
    var income = parseFloat(g_form.getValue('family_income'));
    if (!isNaN(income) && income < 0) {
        errors.push('Annual family income cannot be negative.');
        isValid = false;
    }

    if (!isValid) {
        g_form.addErrorMessage('Please fix the following before submitting:<br>' + errors.join('<br>'));
        return false; // Prevents form submission
    }

    return true; // Allows submission
}
```

### Mandatory Field Management Based on Scholarship Type

```javascript
// Client Script: Dynamic Required Fields
// Table: sn_scholar_application | Type: onChange | Field: scholarship

function onChange(control, oldValue, newValue, isLoading) {
    if (!newValue) return;

    var ga = new GlideAjax('ScholarUtils');
    ga.addParam('sysparm_name', 'getScholarshipDetails');
    ga.addParam('sysparm_scholarship_id', newValue);
    ga.getXML(function(response) {
        var raw = response.responseXML.documentElement.getAttribute('answer');
        if (!raw) return;

        var data;
        try { data = JSON.parse(raw); } catch(e) { return; }

        var isNeedBased = (data.type === 'need_based' || data.type === 'combined');
        var isMerit     = (data.type === 'merit'      || data.type === 'combined');

        // Show/hide AND set mandatory dynamically
        g_form.setVisible('family_income', isNeedBased);
        g_form.setMandatory('family_income', isNeedBased);

        g_form.setVisible('honor_rank', isMerit);
        // honor_rank is optional even for merit — don't setMandatory(true)

        // Clear values when hiding to prevent hidden-field data
        if (!isNeedBased) {
            g_form.setValue('family_income', '');
        }
    });
}
```

### g_form Validation API Quick Reference

```javascript
// Show inline message under a specific field
g_form.showFieldMsg('field_name', 'Message text', 'error');    // red
g_form.showFieldMsg('field_name', 'Message text', 'warning');  // orange
g_form.showFieldMsg('field_name', 'Message text', 'info');     // blue

// Hide inline message
g_form.hideFieldMsg('field_name');

// Page-level messages (banner at top of form)
g_form.addErrorMessage('Error message');       // red banner
g_form.addInfoMessage('Info message');         // blue banner
g_form.hideErrorMessages();                    // clear all banners

// Control field state
g_form.setMandatory('field_name', true);       // mark as required (shows red asterisk)
g_form.setReadOnly('field_name', true);        // disable input
g_form.setDisplay('field_name', false);        // completely hide (no space taken)
g_form.setVisible('field_name', false);        // hide (space reserved)

// Return false from onSubmit to BLOCK the form save
return false;
```

---

## Layer 2 — Server-Side Validation (Business Rules)

### Pattern: Full Before-Insert Validation

```javascript
// Business Rule: sn_scholar_validate_application
// Table: sn_scholar_application
// When: Before Insert
// Condition: current.status == 'submitted'

(function executeRule(current, previous) {
    var errors = [];

    // 1. GWA validation
    var gwa = parseFloat(current.getValue('gwa'));
    if (isNaN(gwa) || gwa < 0 || gwa > 100) {
        errors.push('GWA must be a number between 0 and 100.');
    }

    // 2. Income validation
    var income = parseFloat(current.getValue('family_income'));
    if (isNaN(income) || income < 0) {
        errors.push('Annual family income must be a positive number.');
    }

    // 3. Required fields
    var required = ['gwa', 'family_income', 'course', 'year_level', 'school_type'];
    for (var i = 0; i < required.length; i++) {
        if (!current.getValue(required[i])) {
            errors.push(required[i].replace(/_/g,' ') + ' is required.');
        }
    }

    // 4. Scholarship exists and is open
    var scholarshipId = current.getValue('scholarship');
    var schGR = new GlideRecord('sn_scholar_scholarship');
    if (!schGR.get(scholarshipId)) {
        errors.push('Scholarship not found.');
    } else {
        if (schGR.getValue('status') !== 'open') {
            errors.push('This scholarship is no longer accepting applications.');
        }
        // Check deadline
        var deadline = new GlideDateTime(schGR.getValue('application_deadline'));
        var now = new GlideDateTime();
        if (now.after(deadline)) {
            errors.push('The application deadline has passed.');
        }
        // Check slots remaining
        var slotsRemaining = parseInt(schGR.getValue('slots_remaining') || schGR.getValue('slots_available'));
        if (slotsRemaining <= 0) {
            errors.push('No slots remaining for this scholarship.');
        }
    }

    // 5. Duplicate application check
    var dupGR = new GlideRecord('sn_scholar_application');
    dupGR.addQuery('applicant', current.getValue('applicant'));
    dupGR.addQuery('scholarship', scholarshipId);
    dupGR.addQuery('status', 'NOT IN', 'denied');
    dupGR.setLimit(1);
    dupGR.query();
    if (dupGR.hasNext()) {
        errors.push('You already have an active application for this scholarship.');
    }

    // 6. If errors: abort and show all errors at once
    if (errors.length > 0) {
        gs.addErrorMessage('Your application could not be submitted:\n• ' + errors.join('\n• '));
        current.setAbortAction(true);
    }

})(current, previous);
```

### Pattern: Field Transition Validation (Before Update)

```javascript
// Validates that status can only move in allowed directions
// Business Rule: sn_scholar_validate_status_transition
// When: Before Update | Condition: current.status.changes()

(function executeRule(current, previous) {
    var from = previous.getValue('status');
    var to   = current.getValue('status');

    // Define allowed transitions
    var allowed = {
        'draft':        ['submitted'],
        'submitted':    ['under_review', 'draft'],
        'under_review': ['approved', 'denied', 'draft'],
        'approved':     [],         // terminal — no further changes
        'denied':       []          // terminal — no further changes
    };

    var validNext = allowed[from] || [];
    if (validNext.indexOf(to) === -1) {
        gs.addErrorMessage('Invalid status transition from "' + from + '" to "' + to + '".');
        current.setAbortAction(true);
    }

})(current, previous);
```

### Pattern: Document Completeness Check

```javascript
// Used inside the validate_documents BR
function validateRequiredDocuments(applicationSysId, scholarshipSysId) {
    var schGR = new GlideRecord('sn_scholar_scholarship');
    if (!schGR.get(scholarshipSysId)) return ['Scholarship not found.'];

    var required = (schGR.getValue('required_documents') || '').split(',');
    var missing = [];

    for (var i = 0; i < required.length; i++) {
        var docType = required[i].trim();
        if (!docType) continue;

        var docGR = new GlideRecord('sn_scholar_document');
        docGR.addQuery('application', applicationSysId);
        docGR.addQuery('document_type', docType);
        docGR.addQuery('status', 'NOT IN', 'pending');
        docGR.setLimit(1);
        docGR.query();

        if (!docGR.hasNext()) {
            // Format: "PSA Birth Certificate" from "psa_birth_cert"
            var label = docType.replace(/_/g, ' ')
                               .replace(/\b\w/g, function(c){ return c.toUpperCase(); });
            missing.push(label);
        }
    }

    return missing; // empty array = all docs present
}
```

---

## Layer 3 — Flow & Integration Validation

### Flow Designer Condition Steps (Gate Checks)

```
After "Receive Application":
  Add DECISION step: "Is application complete?"
  Condition: application.status == 'submitted' AND application.gwa != null
  YES branch → proceed to Review
  NO branch  → Send "Incomplete Application" notification → End

After "Provider Decision":
  Add DECISION step: "Is decision valid?"
  Condition: application.approval_action IN ['approve','deny','request_docs']
  YES branch → proceed to branch logic
  NO branch  → Log error → Wait again (handles timeout case)
```

### Validate Compliance Submission (Before Creating Record)

```javascript
// Script Include method called from Flow or widget server script
validateComplianceSubmission: function(awardSysId, semester, academicYear) {
    var errors = [];

    // Award must be active
    var awardGR = new GlideRecord('sn_scholar_award');
    if (!awardGR.get(awardSysId)) {
        errors.push('Award not found.');
        return errors;
    }
    if (awardGR.getValue('status') !== 'active') {
        errors.push('Award is not active. Current status: ' + awardGR.getValue('status'));
    }

    // No duplicate compliance for same semester
    var dupGR = new GlideRecord('sn_scholar_compliance');
    dupGR.addQuery('award', awardSysId);
    dupGR.addQuery('semester', semester);
    dupGR.addQuery('academic_year', academicYear);
    dupGR.setLimit(1);
    dupGR.query();
    if (dupGR.hasNext()) {
        errors.push('Compliance for ' + semester + ' ' + academicYear + ' has already been submitted.');
    }

    return errors; // empty = valid
},
```

---

## ScholarHub Validation Rules Summary

| Field | Rule | Layer | Error Message |
|-------|------|-------|---------------|
| `gwa` | 0–100, not empty | Client + Server | "GWA must be between 0 and 100" |
| `gwa` | >= scholarship.gwa_requirement | Client (advisory) | "Below minimum — you may still apply" |
| `family_income` | > 0, not empty | Client + Server | "Income must be a positive amount" |
| `application_deadline` | < today | Server | "Application deadline has passed" |
| `slots_remaining` | > 0 | Server | "No slots remaining for this scholarship" |
| Required documents | All present (from scholarship.required_documents) | Server | "Missing: [list]" |
| Duplicate application | Only 1 non-denied per scholarship per applicant | Server | "You already have an active application" |
| Status transitions | Only allowed paths | Server | "Invalid status transition" |
| Compliance duplicate | 1 per semester per award | Server | "Already submitted for this semester" |
| Award active | Must be active for compliance | Server | "Award is not active" |

---

## Common Validation Mistakes

```
BAD:  Only validate on the client — user can bypass by API call
BAD:  Only validate on the server — user gets error after filling the whole form
BAD:  Validate in the wrong trigger (After Insert can't use setAbortAction)
BAD:  Show a generic "Error occurred" message — tell the user specifically what is wrong
BAD:  Call gs.addErrorMessage without setAbortAction — message shows but record still saves
GOOD: Validate client for instant UX + server for data integrity — both always
GOOD: Collect ALL errors and show them at once (not one at a time)
GOOD: Before Insert BR for new records, Before Update BR for changes
GOOD: gs.addErrorMessage + setAbortAction + return — always in that order
```
