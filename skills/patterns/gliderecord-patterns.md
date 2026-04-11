# GlideRecord Patterns — ServiceNow Server-Side API
> Agent: use these exact patterns. Do not invent variations. Do not use GlideRecord in Client Scripts.

---

## Basic Query

```javascript
// Query records with conditions
var gr = new GlideRecord('sn_scholar_application');
gr.addQuery('status', 'submitted');
gr.addQuery('scholarship', scholarshipSysId);
gr.orderByDesc('submission_date');
gr.query();
while (gr.next()) {
    var appNumber = gr.getValue('number');
    var applicantName = gr.getDisplayValue('applicant');
    var gwa = parseFloat(gr.getValue('gwa'));
}
```

## Get Single Record

```javascript
// By sys_id
var gr = new GlideRecord('sn_scholar_scholarship');
if (gr.get(sysId)) {
    var name = gr.getValue('name');
    var type = gr.getValue('scholarship_type');
} else {
    gs.addErrorMessage('Scholarship not found');
}

// By field value (gets first match)
var gr = new GlideRecord('sn_scholar_application');
gr.addQuery('number', 'APP0001001');
gr.query();
if (gr.next()) {
    // record found
}
```

## Create Record

```javascript
var gr = new GlideRecord('sn_scholar_award');
gr.initialize();
gr.setValue('application', applicationSysId);
gr.setValue('scholar', scholarSysId);
gr.setValue('scholarship', scholarshipSysId);
gr.setValue('coverage_type', 'full_tuition');
gr.setValue('status', 'active');
gr.setValue('disbursement_status', 'pending');
gr.setValue('start_date', new GlideDate().getLocalDate());
var awardSysId = gr.insert();
// awardSysId is the sys_id of the new record, or null on failure
if (!awardSysId) {
    gs.addErrorMessage('Failed to create award record');
}
```

## Update Record

```javascript
var gr = new GlideRecord('sn_scholar_application');
if (gr.get(applicationSysId)) {
    gr.setValue('status', 'approved');
    gr.setValue('review_notes', reviewNotes);
    gr.update();
}
```

## Delete Record (Rare — use with caution)

```javascript
var gr = new GlideRecord('sn_scholar_document');
if (gr.get(docSysId)) {
    gr.deleteRecord();
}
```

---

## Query Operators

```javascript
// NOT equal
gr.addQuery('status', '!=', 'denied');

// IN a list
gr.addQuery('status', 'IN', 'submitted,under_review');

// NOT IN a list
gr.addQuery('status', 'NOT IN', 'denied,draft');

// Greater than / Less than
gr.addQuery('gwa', '>=', 85.00);
gr.addQuery('application_deadline', '>=', gs.nowDateTime());

// LIKE (contains)
gr.addQuery('course', 'LIKE', 'BS');

// NULL check
gr.addNullQuery('reviewer');      // reviewer IS NULL
gr.addNotNullQuery('submission_date'); // submission_date IS NOT NULL

// OR condition
var qc = gr.addQuery('status', 'submitted');
qc.addOrCondition('status', 'under_review');

// Encoded query (copy from list view filter in ServiceNow)
gr.addEncodedQuery('status=submitted^scholarship=' + scholarshipSysId);
```

---

## Getting Values Correctly

```javascript
var gr = new GlideRecord('sn_scholar_application');
gr.get(sysId);

// String fields — use getValue()
gr.getValue('status');           // returns 'submitted' (choice value)
gr.getDisplayValue('status');    // returns 'Submitted' (display label)

// Reference fields — getValue() returns sys_id, getDisplayValue() returns name
gr.getValue('applicant');        // sys_id of the user
gr.getDisplayValue('applicant'); // user's full name
gr.scholarship.name.toString();  // ALSO works but use getValue for consistency

// Date/Time fields
gr.getValue('submission_date');           // '2026-05-12 08:00:00'
gr.getDisplayValue('submission_date');    // '05/12/2026 08:00:00' (formatted)

// Numeric fields
var gwa = parseFloat(gr.getValue('gwa'));  // always parse to use in math
var income = parseFloat(gr.getValue('family_income'));

// Boolean fields
var isActive = gr.getValue('active') == 'true';
```

---

## Limit and Count

```javascript
// Limit results
gr.setLimit(10);
gr.query();

// Count without fetching all records
var ga = new GlideAggregate('sn_scholar_application');
ga.addQuery('scholarship', scholarshipSysId);
ga.addQuery('status', 'submitted');
ga.addAggregate('COUNT');
ga.query();
var count = 0;
if (ga.next()) {
    count = parseInt(ga.getAggregate('COUNT'));
}
```

---

## Working with Dates

```javascript
// Current date and time
gs.nowDateTime()        // '2026-05-12 08:00:00' UTC
new GlideDateTime()     // GlideDateTime object for current time
new GlideDate().getLocalDate() // Today's date as string

// Compare dates
var deadline = new GlideDateTime(gr.getValue('application_deadline'));
var now = new GlideDateTime();
if (now.after(deadline)) {
    // deadline has passed
}

// Date arithmetic
var gdt = new GlideDateTime();
gdt.addDaysUTC(30);  // 30 days from now
var futureDate = gdt.getDate().getLocalDate();
```

---

## Business Rule Specific Patterns

```javascript
// In a Business Rule, use `current` for the record being processed
// For Before Insert:
current.getValue('status')          // read current field
current.setValue('status', 'submitted') // modify before save
current.setAbortAction(true)         // STOP the insert/update
gs.addErrorMessage('Error message visible to user')
gs.addInfoMessage('Info message visible to user')

// For After Insert/Update:
current.getValue('sys_id')  // the new record's sys_id is available
current.isNewRecord()       // true for new inserts

// `previous` object (only available in Before/After Update):
previous.getValue('status') // what the status was before the update

// Check if a specific field changed:
if (current.status.changes()) {
    var oldStatus = previous.getValue('status');
    var newStatus = current.getValue('status');
}
```

---

## ScholarHub-Specific Common Queries

```javascript
// Get scholarship requirements as array
var gr = new GlideRecord('sn_scholar_scholarship');
gr.get(scholarshipSysId);
var reqDocs = gr.getValue('required_documents'); // "psa_birth_cert,report_card,itr"
var docArray = reqDocs ? reqDocs.split(',') : [];

// Check if applicant has submitted a specific document type
var docGR = new GlideRecord('sn_scholar_document');
docGR.addQuery('application', applicationSysId);
docGR.addQuery('document_type', 'psa_birth_cert');
docGR.addQuery('status', '!=', 'pending');
docGR.query();
var hasPSA = docGR.hasNext();

// Get active award for a scholar
var awardGR = new GlideRecord('sn_scholar_award');
awardGR.addQuery('scholar', scholarSysId);
awardGR.addQuery('status', 'active');
awardGR.setLimit(1);
awardGR.query();
if (awardGR.next()) {
    var awardSysId = awardGR.getUniqueValue();
}

// Decrement slots_remaining on approval
var schGR = new GlideRecord('sn_scholar_scholarship');
if (schGR.get(scholarshipSysId)) {
    var remaining = parseInt(schGR.getValue('slots_remaining')) - 1;
    schGR.setValue('slots_remaining', Math.max(0, remaining));
    schGR.update();
}
```

---

## NEVER Do These

```javascript
// NEVER use GlideRecord in a Client Script (security violation)
// Use GlideAjax instead — see glideajax-patterns.md

// NEVER modify current in an After Business Rule (too late, record already saved)
// Use Before BR to modify values before save

// NEVER use gr.name (dot notation) for string comparison — it returns a GlideElement object
if (gr.status == 'submitted') { ... }  // WRONG — always false
if (gr.getValue('status') == 'submitted') { ... }  // CORRECT

// NEVER forget to call .query() before .next()
gr.addQuery('status', 'submitted');
// gr.next() here — WRONG, query() not called
gr.query();
gr.next(); // CORRECT
```
