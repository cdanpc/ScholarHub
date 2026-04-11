# Error Handling Patterns — ScholarHub
> Agent: errors happen at every layer. Handle them where they occur. Never let an unhandled error reach the user as a blank screen or raw stack trace.

---

## Error Handling by Layer

```
Layer               Where It Happens          How to Handle
──────────────────  ────────────────────────  ─────────────────────────────────────────
Client Script       Browser / g_form          g_form.addErrorMessage, return false
Business Rule       Server / database layer   gs.addErrorMessage + setAbortAction
Script Include      Server / called by others try/catch + return error object
Widget server script ServiceNow SP server     try/catch + set data.error flag
Flow Designer       Workflow engine           Error lane + fault conditions
Integration Hub     External API calls        HTTP status check + fallback data
```

---

## Business Rule Error Handling

```javascript
// Correct pattern: always abort + message + return
(function executeRule(current, previous) {
    try {
        var scholarshipId = current.getValue('scholarship');
        var schGR = new GlideRecord('sn_scholar_scholarship');

        if (!schGR.get(scholarshipId)) {
            gs.addErrorMessage('The selected scholarship could not be found. Please refresh and try again.');
            current.setAbortAction(true);
            return;
        }

        if (schGR.getValue('status') !== 'open') {
            gs.addErrorMessage('This scholarship is no longer accepting applications.');
            current.setAbortAction(true);
            return;
        }

        // Main logic...

    } catch (e) {
        // Catch unexpected runtime errors
        gs.log('ScholarHub BR Error [' + current.getTableName() + ']: ' + e.message, 'ScholarHub');
        gs.addErrorMessage('An unexpected error occurred. Please contact your administrator. (Ref: ' + current.getUniqueValue() + ')');
        current.setAbortAction(true);
    }
})(current, previous);
```

**Rules:**
- Always `setAbortAction(true)` AND `return` — never one without the other
- Always log the raw error to sys_log AND show a clean message to the user
- Never show technical details (table names, sys_ids, stack traces) to the user
- Include a reference ID so admins can find the log entry

---

## Script Include Error Handling

```javascript
// Pattern: return an object with { success, data, error }
// Never throw — the caller may not catch

var ScholarUtils = Class.create();
ScholarUtils.prototype = Object.extendsObject(AbstractAjaxProcessor, {

    // For GlideAjax calls — return string that client parses
    getScholarshipDetails: function() {
        var id = this.getParameter('sysparm_scholarship_id');

        if (!id) {
            return JSON.stringify({ success: false, error: 'No scholarship ID provided' });
        }

        var gr = new GlideRecord('sn_scholar_scholarship');
        if (!gr.get(id)) {
            return JSON.stringify({ success: false, error: 'Scholarship not found' });
        }

        return JSON.stringify({
            success: true,
            data: {
                type:           gr.getValue('scholarship_type'),
                gwa_req:        gr.getValue('gwa_requirement'),
                income_cap:     gr.getValue('income_cap'),
                required_docs:  gr.getValue('required_documents'),
                status:         gr.getValue('status')
            }
        });
    },

    // For server-to-server calls — return result object
    createAwardRecord: function(applicationSysId) {
        try {
            var appGR = new GlideRecord('sn_scholar_application');
            if (!appGR.get(applicationSysId)) {
                return { success: false, error: 'Application ' + applicationSysId + ' not found' };
            }

            var awardGR = new GlideRecord('sn_scholar_award');
            awardGR.initialize();
            awardGR.setValue('application', applicationSysId);
            awardGR.setValue('scholar', appGR.getValue('applicant'));
            awardGR.setValue('scholarship', appGR.getValue('scholarship'));
            awardGR.setValue('status', 'active');
            awardGR.setValue('disbursement_status', 'pending');
            var awardId = awardGR.insert();

            if (!awardId) {
                return { success: false, error: 'GlideRecord insert failed' };
            }

            return { success: true, awardSysId: awardId };

        } catch (e) {
            gs.log('ScholarHub: createAwardRecord failed — ' + e.message, 'ScholarHub');
            return { success: false, error: e.message };
        }
    },

    type: 'ScholarUtils'
});
```

**Calling the result object:**
```javascript
var util = new ScholarUtils();
var result = util.createAwardRecord(applicationSysId);

if (!result.success) {
    gs.addErrorMessage('Could not create award: ' + result.error);
    current.setAbortAction(true);
    return;
}

var awardId = result.awardSysId;
```

---

## Widget Server Script Error Handling

```javascript
// Widget server script — always wrap in try/catch, always set error state on data
(function() {
    data.error = null;    // null = no error
    data.items = [];

    try {
        var gr = new GlideRecord('sn_scholar_application');
        gr.addQuery('applicant', gs.getUserID());
        gr.orderByDesc('submission_date');
        gr.query();

        while (gr.next()) {
            data.items.push({
                sys_id:       gr.getUniqueValue(),
                number:       gr.getValue('number'),
                scholarship:  gr.getDisplayValue('scholarship'),
                status:       gr.getValue('status'),
                status_label: gr.getDisplayValue('status'),
                submitted:    gr.getDisplayValue('submission_date')
            });
        }

        if (data.items.length === 0) {
            data.emptyMessage = 'You have not submitted any applications yet.';
        }

    } catch (e) {
        gs.log('ScholarHub widget [my-applications] error: ' + e.message, 'ScholarHub');
        data.error = 'Could not load your applications. Please refresh the page.';
    }
})();
```

**In the HTML template — always handle the error state:**
```html
<!-- Show error state -->
<div class="sh-alert sh-alert-error" ng-if="c.data.error">
  {{c.data.error}}
  <button class="sh-btn sh-btn-ghost sh-btn-sm" ng-click="c.reload()">Try again</button>
</div>

<!-- Show empty state -->
<div class="sh-empty" ng-if="!c.data.error && c.data.items.length === 0">
  <p class="sh-empty-title">No applications yet</p>
  <p class="sh-empty-body">{{c.data.emptyMessage}}</p>
</div>

<!-- Show content -->
<div ng-if="!c.data.error && c.data.items.length > 0">
  <!-- actual content -->
</div>
```

**In the client controller — reload function:**
```javascript
function($scope, spUtil) {
  var c = this;

  c.reload = function() {
    $scope.$emit('sp.widget.reload');
  };
}
```

---

## Flow Designer Error Handling

### Fault Lane (Error Branch)

Every flow that calls an external service or has a critical step should have a fault lane:

```
Flow: ScholarHub Application Lifecycle

Step: Call Action "AI Eligibility Pre-screen"
  ├── SUCCESS path → continue normal flow
  └── FAULT path   → Set ai_pre_screen_result = '{"status":"error","note":"unavailable"}'
                   → Continue flow WITHOUT blocking (AI is advisory, not blocking)

Step: Send Email notification
  ├── SUCCESS path → next step
  └── FAULT path   → Log error to sys_log → Continue (don't fail the whole flow for email)

Step: Create Award Record (on approval)
  ├── SUCCESS path → send congratulations email
  └── FAULT path   → STOP flow + notify admin + set application status = 'error'
                   → This IS critical — award must be created
```

**Configuring fault lanes in Flow Designer:**
```
On the Action step → click "..." → "Add fault lane"
Fault lane condition: The action threw an exception or returned an error
```

### Critical vs Non-Critical Errors in Flows

```
CRITICAL (fail the flow, notify admin):
  - Award record creation fails
  - Application status update fails
  - Database insert/update returns null

NON-CRITICAL (log and continue):
  - AI pre-screen API timeout
  - Email notification send fails (record the failure, don't block approval)
  - Slot counter decrement fails (can be corrected manually)
```

### Flow Error Logging

```javascript
// In a Script step within a Flow fault lane:
var context = fd_data; // Flow Designer data context
gs.log(
    'ScholarHub Flow ERROR: ' + context.flowName +
    ' | Application: ' + (context.applicationSysId || 'unknown') +
    ' | Error: ' + context.errorMessage,
    'ScholarHub'
);
// Create an incident or task for admin to review (optional but professional)
```

---

## Integration Hub / API Error Handling

```javascript
// Full pattern in widget server script calling Claude API
try {
    var rm = new sn_ws.RESTMessageV2('claude_messages', 'check_eligibility');
    rm.setStringParameterNoEscape('system_prompt', systemPrompt);
    rm.setStringParameterNoEscape('user_message', userMessage);
    rm.setStringParameterNoEscape('api_key', gs.getProperty('x_snc_scholar.claude_api_key'));
    rm.setHttpTimeout(10000); // 10 second timeout — don't hang forever

    var response = rm.execute();
    var httpCode = response.getStatusCode();
    var body     = response.getBody();

    if (httpCode === 200) {
        var parsed = JSON.parse(body);
        var resultText = parsed.content[0].text.replace(/```json|```/g, '').trim();
        var result = JSON.parse(resultText);
        data.aiStatus  = result.status  || 'error';
        data.aiMatched = result.matched || [];
        data.aiMissing = result.missing || [];
        data.aiNote    = result.note    || '';

    } else if (httpCode === 429) {
        // Rate limited — don't log as error, it's expected
        data.aiStatus = 'unavailable';
        data.aiNote   = 'Eligibility check is busy. You may still apply manually.';

    } else if (httpCode >= 500) {
        gs.log('ScholarHub AI: API server error ' + httpCode + ' — ' + body.substring(0,200), 'ScholarHub');
        data.aiStatus = 'unavailable';
        data.aiNote   = 'Eligibility check is temporarily unavailable.';

    } else {
        gs.log('ScholarHub AI: Unexpected HTTP ' + httpCode + ' — ' + body.substring(0,200), 'ScholarHub');
        data.aiStatus = 'unavailable';
        data.aiNote   = 'Could not complete eligibility check.';
    }

} catch (e) {
    if (e.message && e.message.indexOf('timeout') !== -1) {
        data.aiStatus = 'unavailable';
        data.aiNote   = 'Eligibility check timed out. You may still apply manually.';
    } else {
        gs.log('ScholarHub AI: Exception — ' + e.message, 'ScholarHub');
        data.aiStatus = 'unavailable';
        data.aiNote   = 'Could not complete eligibility check. You may still apply.';
    }
}
```

---

## Logging Strategy

```javascript
// ALWAYS include: system prefix + context + message
// Format: gs.log('[PREFIX]: [CONTEXT] — [MESSAGE]', '[SOURCE]')

// Info (normal operation worth noting)
gs.log('ScholarHub: Award created — ' + awardNumber + ' for application ' + appNumber, 'ScholarHub');

// Warning (something unexpected but recoverable)
gs.log('ScholarHub WARN: Slot counter negative on scholarship ' + scholarshipNumber + ' — corrected to 0', 'ScholarHub');

// Error (something failed — include enough context to debug)
gs.log('ScholarHub ERROR: Award creation failed for application ' + appNumber + ' — ' + errorMessage, 'ScholarHub');

// Never log sensitive data
gs.log('ScholarHub: Processing application for [redacted]', 'ScholarHub'); // not the applicant's name/email

// Viewing logs:
// Navigate to: /syslog_list.do?sysparm_query=source=ScholarHub
```

**Log levels in ServiceNow sys_log:**
- `gs.log(message, source)` — level 0 (debug)
- `gs.info(message)` — level 0 (info)
- `gs.warn(message)` — level 1 (warning)
- `gs.error(message)` — level 2 (error) — appears in default sys_log filter

---

## User-Facing Error Messages — Writing Guide

```
BAD:  "Error: GlideRecord.insert() returned null at line 47"  (technical jargon)
BAD:  "An error occurred."  (useless)
BAD:  "Please try again."  (no context)

GOOD: "Your application could not be submitted. Missing documents: PSA Birth Certificate, ITR."
GOOD: "This scholarship is no longer accepting applications."
GOOD: "Could not load your applications. Please refresh the page. If the issue persists, contact support."

Rules:
1. Tell the user WHAT went wrong (not HOW it failed technically)
2. Tell the user WHAT TO DO NEXT (refresh, contact admin, correct field)
3. If it's a validation error, list ALL issues at once (not one by one)
4. For unexpected system errors, give an admin reference ID (the record sys_id)
5. Never expose table names, sys_ids, or code line numbers in user-facing messages
```

---

## Error State Design in Widgets

```html
<!-- Three states every widget must handle: loading, error, empty, content -->

<!-- Loading -->
<div class="sh-loading" ng-if="c.loading">
  <div class="sh-spinner"></div>
  <p class="sh-caption">Loading...</p>
</div>

<!-- Error -->
<div class="sh-alert sh-alert-error" ng-if="!c.loading && c.data.error">
  {{c.data.error}}
</div>

<!-- Empty -->
<div class="sh-empty" ng-if="!c.loading && !c.data.error && c.data.items.length === 0">
  <p class="sh-empty-title">No results</p>
  <p class="sh-empty-body">{{c.data.emptyMessage || 'Nothing to show here yet.'}}</p>
</div>

<!-- Content -->
<div ng-if="!c.loading && !c.data.error && c.data.items.length > 0">
  <!-- content here -->
</div>
```

```css
/* Loading spinner using CSS only — no image dependency */
.sh-loading {
  text-align: center; padding: 40px;
  color: var(--sh-text-muted);
}
.sh-spinner {
  width: 24px; height: 24px; margin: 0 auto 12px;
  border: 2.5px solid var(--sh-border);
  border-top-color: var(--sh-brand);
  border-radius: 50%;
  animation: sh-spin 0.7s linear infinite;
}
@keyframes sh-spin { to { transform: rotate(360deg); } }
```
