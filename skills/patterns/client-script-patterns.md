# Client Script Patterns — ServiceNow
> Agent: Client Scripts run in the BROWSER. Never use GlideRecord here. Use GlideAjax for server calls.

---

## Client Script Types

| Type | Fires When | Use For |
|------|-----------|---------|
| `onLoad` | Form opens | Hide/show fields, set defaults, fetch initial data |
| `onChange` | Specific field value changes | React to field changes, validate on the fly |
| `onSubmit` | User submits the form | Final validation before save |
| `onCellEdit` | Cell edited in list view | (Not used in ScholarHub) |

---

## g_form API Reference

```javascript
// Read field values
g_form.getValue('field_name')           // string value
g_form.getDisplayValue('field_name')    // display value (for reference fields)
g_form.getReference('field_name', callback)  // async fetch of referenced record

// Set field values
g_form.setValue('field_name', 'value')
g_form.setDisplayValue('field_name', 'Display Label')

// Show / Hide fields
g_form.setVisible('field_name', true)   // show
g_form.setVisible('field_name', false)  // hide

// Enable / Disable (read-only)
g_form.setReadOnly('field_name', true)  // make read-only
g_form.setReadOnly('field_name', false) // make editable

// Mandatory
g_form.setMandatory('field_name', true)  // make required
g_form.setMandatory('field_name', false) // make optional

// Add messages
g_form.addInfoMessage('Info message text')   // blue banner
g_form.addErrorMessage('Error message text') // red banner
g_form.hideErrorMessages()                   // clear messages

// Check if field exists on form
g_form.hasField('field_name')  // true/false
```

---

## ScholarHub Client Scripts

### CS 1: Show Income Fields for Need-Based

```javascript
// Table: sn_scholar_application
// Type: onChange
// Field name: scholarship
// Script:

function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading) return;
    if (!newValue) return;

    // Fetch the scholarship type from the server
    var ga = new GlideAjax('ScholarUtils');
    ga.addParam('sysparm_name', 'getScholarshipType');
    ga.addParam('sysparm_scholarship_id', newValue);
    ga.getXML(function(response) {
        var answer = response.responseXML.documentElement.getAttribute('answer');
        var isNeedBased = (answer == 'need_based' || answer == 'combined');

        // Need-based fields
        g_form.setVisible('family_income', isNeedBased);
        g_form.setMandatory('family_income', isNeedBased);

        // Need-based document fields (if rendered as form fields)
        g_form.setVisible('itr_field', isNeedBased);
        g_form.setVisible('indigency_field', isNeedBased);
        g_form.setVisible('house_photo_field', isNeedBased);
        g_form.setVisible('barangay_cert_field', isNeedBased);

        // Merit-based fields
        var isMerit = (answer == 'merit' || answer == 'combined');
        g_form.setVisible('honor_rank', isMerit);
        g_form.setVisible('recommendation_field', isMerit);
    });
}
```

### CS 2: GWA Eligibility Indicator

```javascript
// Table: sn_scholar_application
// Type: onChange
// Field name: gwa

function onChange(control, oldValue, newValue, isLoading) {
    if (isLoading) return;

    var gwa = parseFloat(newValue);
    var scholarshipId = g_form.getValue('scholarship');
    if (!scholarshipId || isNaN(gwa)) return;

    var ga = new GlideAjax('ScholarUtils');
    ga.addParam('sysparm_name', 'getGWARequirement');
    ga.addParam('sysparm_scholarship_id', scholarshipId);
    ga.getXML(function(response) {
        var required = parseFloat(response.responseXML.documentElement.getAttribute('answer'));
        if (isNaN(required)) return;

        g_form.hideErrorMessages();
        if (gwa >= required) {
            g_form.addInfoMessage('GWA ' + gwa + ' meets the minimum requirement of ' + required);
        } else {
            g_form.addErrorMessage('GWA ' + gwa + ' is below the minimum requirement of ' + required +
                '. You may still apply, but check your eligibility.');
        }
    });
}
```

### CS 3: Lock Form After Decision

```javascript
// Table: sn_scholar_application
// Type: onLoad

function onLoad() {
    var status = g_form.getValue('status');
    var lockedStatuses = ['approved', 'denied'];

    if (lockedStatuses.indexOf(status) >= 0) {
        var fields = ['gwa', 'family_income', 'course', 'year_level',
                      'school_type', 'honor_rank', 'scholarship'];
        for (var i = 0; i < fields.length; i++) {
            g_form.setReadOnly(fields[i], true);
        }
        // Hide Submit button (UI Action)
        g_form.hideRelatedList('Submit Application');

        if (status == 'approved') {
            g_form.addInfoMessage('This application has been approved.');
        } else {
            g_form.addErrorMessage('This application was not approved.');
        }
    }
}
```

---

## GlideAjax Pattern (Server Call from Client)

```javascript
// Client Script calls a Script Include method:
var ga = new GlideAjax('ScholarUtils');        // Script Include name
ga.addParam('sysparm_name', 'methodName');     // method to call (must be allowed)
ga.addParam('sysparm_param1', value1);         // parameters
ga.addParam('sysparm_param2', value2);
ga.getXML(callbackFunction);                   // ALWAYS async

function callbackFunction(response) {
    var answer = response.responseXML.documentElement.getAttribute('answer');
    // 'answer' is the string returned by the Script Include method
    // Use the answer here
}
```

**The Script Include must extend AbstractAjaxProcessor:**
```javascript
// Script Include: ScholarUtils
// Client callable: true
var ScholarUtils = Class.create();
ScholarUtils.prototype = Object.extendsObject(AbstractAjaxProcessor, {
    getScholarshipType: function() {
        var id = this.getParameter('sysparm_scholarship_id');
        var gr = new GlideRecord('sn_scholar_scholarship');
        if (gr.get(id)) {
            return gr.getValue('scholarship_type');
        }
        return '';
    },
    getGWARequirement: function() {
        var id = this.getParameter('sysparm_scholarship_id');
        var gr = new GlideRecord('sn_scholar_scholarship');
        if (gr.get(id)) {
            return gr.getValue('gwa_requirement') || '0';
        }
        return '0';
    },
    type: 'ScholarUtils'
});
```
