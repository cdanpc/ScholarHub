# ACL Patterns — ServiceNow Access Control
> Agent: ACLs protect data. A wrong ACL exposes records to wrong roles or locks out legitimate users. Test all three roles after every change.

---

## ACL Record Fields

| Field | Description |
|-------|-------------|
| Type | `record` (row-level), `field` (field-level), `*` (all types) |
| Name | `tablename.*` for record ACL, `tablename.fieldname` for field ACL |
| Operation | `read`, `write`, `create`, `delete` |
| Role | Which role this rule applies to (leave blank = applies to all) |
| Active | Must be true |
| Script | JavaScript returning `answer = true/false` |
| Condition | Simple condition (alternative to script for simple cases) |

---

## ScholarHub ACL Scripts

### Read own applications (applicant)

```javascript
// Table: sn_scholar_application
// Operation: read
// Role: sn_scholar.applicant

answer = current.applicant == gs.getUserID();
```

### Read applications for own scholarships (provider)

```javascript
// Table: sn_scholar_application
// Operation: read
// Role: sn_scholar.provider

answer = current.scholarship.provider == gs.getUserID();
```

### Write own scholarship only (provider)

```javascript
// Table: sn_scholar_scholarship
// Operation: write
// Role: sn_scholar.provider

answer = current.provider == gs.getUserID();
```

### Create compliance only if active award exists (applicant)

```javascript
// Table: sn_scholar_compliance
// Operation: create
// Role: sn_scholar.applicant

var award = new GlideRecord('sn_scholar_award');
award.addQuery('scholar', gs.getUserID());
award.addQuery('status', 'active');
award.setLimit(1);
award.query();
answer = award.hasNext();
```

### Admin full access

```javascript
// Table: sn_scholar_* (any table)
// Operation: * (all)
// Role: sn_scholar.admin

answer = true; // admin always gets access
```

---

## ACL Evaluation Order

ServiceNow evaluates ACLs from most specific to least specific:
1. Field-level ACL on a specific field
2. Record-level ACL on a specific table
3. Wildcard ACL (`*.*`)

**Multiple ACLs for same operation**: ALL matching ACLs must return `answer = true`. If any returns false, access is denied.

---

## Testing ACLs

Always test with impersonation:
```
1. Impersonate applicant@scholarhub.test
2. Navigate to the table or portal page
3. Verify: can see own records, cannot see others' records
4. Impersonate provider@scholarhub.test
5. Verify: can see applications for their scholarships, not others'
6. End impersonation
7. Test admin access with your admin account
```

---

# GlideAjax Patterns — Server-Client Communication
> Agent: when a Client Script needs server data, use GlideAjax. Never use GlideRecord in client-side code.

---

## Complete Pattern: Client Script → Script Include

### Step 1: Script Include (server-side)

```javascript
// Name: ScholarUtils
// Client callable: YES (this checkbox must be checked)
// Extends: AbstractAjaxProcessor

var ScholarUtils = Class.create();
ScholarUtils.prototype = Object.extendsObject(AbstractAjaxProcessor, {

    // Get scholarship type
    getScholarshipType: function() {
        var id = this.getParameter('sysparm_scholarship_id');
        var gr = new GlideRecord('sn_scholar_scholarship');
        if (gr.get(id)) {
            return gr.getValue('scholarship_type');
        }
        return '';
    },

    // Get GWA requirement
    getGWARequirement: function() {
        var id = this.getParameter('sysparm_scholarship_id');
        var gr = new GlideRecord('sn_scholar_scholarship');
        if (gr.get(id)) {
            return gr.getValue('gwa_requirement') || '0';
        }
        return '0';
    },

    // Get required document types as JSON string
    getRequiredDocuments: function() {
        var id = this.getParameter('sysparm_scholarship_id');
        var gr = new GlideRecord('sn_scholar_scholarship');
        if (gr.get(id)) {
            var docs = gr.getValue('required_documents') || '';
            return docs; // "psa_birth_cert,report_card,itr"
        }
        return '';
    },

    // Check if user has active award
    hasActiveAward: function() {
        var award = new GlideRecord('sn_scholar_award');
        award.addQuery('scholar', gs.getUserID());
        award.addQuery('status', 'active');
        award.setLimit(1);
        award.query();
        return award.hasNext() ? 'true' : 'false';
    },

    type: 'ScholarUtils'
});
```

### Step 2: Client Script (browser-side)

```javascript
// Call a single method
var ga = new GlideAjax('ScholarUtils');
ga.addParam('sysparm_name', 'getRequiredDocuments');
ga.addParam('sysparm_scholarship_id', g_form.getValue('scholarship'));
ga.getXML(function(response) {
    var result = response.responseXML.documentElement.getAttribute('answer');
    if (result) {
        var docs = result.split(',');
        // Use docs array to render checklist
        for (var i = 0; i < docs.length; i++) {
            var docType = docs[i].trim();
            // show the corresponding document field or generate checklist item
            g_form.setVisible(docType + '_field', true);
        }
    }
});
```

---

## Returning Complex Data from GlideAjax

For more than a single string, return JSON:

```javascript
// Script Include method returning JSON
getScholarshipDetails: function() {
    var id = this.getParameter('sysparm_scholarship_id');
    var gr = new GlideRecord('sn_scholar_scholarship');
    if (gr.get(id)) {
        var result = {
            type: gr.getValue('scholarship_type'),
            gwa: gr.getValue('gwa_requirement'),
            income_cap: gr.getValue('income_cap'),
            docs: gr.getValue('required_documents')
        };
        return JSON.stringify(result);
    }
    return '{}';
},
```

```javascript
// Client Script parsing JSON response
ga.getXML(function(response) {
    var raw = response.responseXML.documentElement.getAttribute('answer');
    try {
        var data = JSON.parse(raw);
        // data.type, data.gwa, data.income_cap, data.docs
    } catch(e) {
        console.warn('ScholarHub: Failed to parse GlideAjax response', e);
    }
});
```

---

## Common GlideAjax Mistakes

- **`ga.getXML()` is always async** — never expect the result to be available on the line after the call
- **Script Include must have "Client callable" checked** — if not checked, the call silently returns empty
- **Parameter names must start with `sysparm_`** — e.g., `sysparm_scholarship_id` not `scholarship_id`
- **The `answer` attribute holds the return value** — access via `getAttribute('answer')`
- **Never call `ga.getXMLWait()`** — it blocks the browser thread and is deprecated
