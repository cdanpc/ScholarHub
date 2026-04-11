# Events, Hooks & In-Portal Notifications — ScholarHub
> Agent: events connect layers without tight coupling. Hooks give you lifecycle control. In-portal notifications give users instant feedback without reloading.

---

## Part 1 — ServiceNow Events System

### What Are Events?

ServiceNow Events are named signals fired by scripts and caught by Script Actions (which run code) or Notifications (which send email). Events decouple the trigger from the action — a Business Rule fires an event, and separately a Notification listens for it.

```
Event flow:
  Script fires event → Event Queue (sys_event) → Script Action runs / Email sent
```

### Firing a Custom Event

```javascript
// In a Business Rule (After Insert/Update) or Script Include:
// gs.eventQueue(eventName, record, param1, param2)

// Fire "scholar.application.approved" event
gs.eventQueue(
    'x_snc_scholar.application.approved',   // event name (use your app scope prefix)
    current,                                  // the GlideRecord that triggered it
    current.getValue('applicant'),            // param1 — available in notification as ${event.parm1}
    current.getDisplayValue('scholarship')    // param2 — available as ${event.parm2}
);

// Fire "scholar.compliance.due" event for a specific scholar
gs.eventQueue(
    'x_snc_scholar.compliance.due',
    awardGR,
    awardGR.getValue('scholar'),
    academicYear + ' — ' + semester
);
```

### ScholarHub Custom Events

Register these in the Event Registry (navigate to: sys_event_registry_list.do):

```
Event Name                              Description                         Fired By
────────────────────────────────────── ─────────────────────────────────── ────────────────────────
x_snc_scholar.application.submitted    Applicant submits application       BR: set_status_on_submit
x_snc_scholar.application.approved     Provider approves application       UA: Approve Application
x_snc_scholar.application.denied       Provider denies application         UA: Deny Application
x_snc_scholar.docs.missing             Provider requests more docs         UA: Request More Docs
x_snc_scholar.award.created            Award record created on approval    BR / Script Include
x_snc_scholar.award.disbursed          Provider releases disbursement      UA: Release Disbursement
x_snc_scholar.compliance.due           Compliance deadline approaching     Scheduled job
x_snc_scholar.compliance.submitted     Scholar submits compliance docs     Widget / form
x_snc_scholar.scholar.non_compliant    Scholar marked non-compliant        BR: flag_non_compliant
```

### Creating an Event Registry Entry

```
Navigate to: System Policy → Events → Registry → New

Name:         x_snc_scholar.application.approved
Table:        sn_scholar_application
Description:  Fired when a provider approves a scholarship application
Suffix:       .approved
```

### Connecting Events to Email Notifications

```
In a Notification record:
  Send when: Event is fired
  Event name: x_snc_scholar.application.approved

  Recipient: Script
    Script:
      answer = event.parm1; // sys_id of the applicant (param1 from gs.eventQueue)

  In the email template body:
    ${event.parm1}  → param1 value (applicant sys_id or name)
    ${event.parm2}  → param2 value (scholarship name)
    ${event.source.number} → the application number
```

### Connecting Events to Script Actions

Script Actions let you run code when an event fires — for non-email reactions:

```
Navigate to: System Policy → Events → Script Actions → New

Name:         ScholarHub — On Application Approved
Event name:   x_snc_scholar.application.approved
Active:       YES

Script:
  // 'event' object is available — event.parm1, event.parm2, event.source
  var applicationId = event.source; // GlideRecord of sn_scholar_application

  // Create award record
  var util = new ScholarUtils();
  var result = util.createAwardRecord(applicationId);

  if (!result.success) {
      gs.log('ScholarHub ScriptAction: Award creation failed — ' + result.error, 'ScholarHub');
  }
```

### Using Events vs Direct Business Rule Logic

```
USE EVENTS WHEN:
  - Multiple things need to happen when one thing occurs
    (approval → create award + send email + decrement slots + update analytics)
  - The reaction is optional or could be extended later
  - You want to keep the Business Rule lean

USE DIRECT BR LOGIC WHEN:
  - The action MUST complete before the record saves (validation, abort)
  - The action is tightly coupled to the record save (stamp a date, set a field)
  - Performance matters and event queue latency is unacceptable
```

---

## Part 2 — Angular Hooks (Service Portal Widget Lifecycle)

### The Four Lifecycle Hooks

```javascript
function($scope, $http, spUtil) {
    var c = this;

    // ── $onInit ────────────────────────────────────────────────────
    // Fires ONCE when the widget is first loaded.
    // Use for: initial data fetch, setup, reading options, checking user role.
    // The server data (c.data) is already populated when $onInit runs.

    c.$onInit = function() {
        // Load additional dynamic data not available from server script
        c.currentFilter = 'all';
        c.loading = false;

        // Read widget options (set in portal designer)
        c.title = c.options.title || 'Scholarships';

        // Confirm server data loaded correctly
        if (!c.data.scholarships) {
            c.data.error = 'Could not load scholarships. Please refresh.';
        }
    };

    // ── $postLink ──────────────────────────────────────────────────
    // Fires AFTER the HTML template is rendered and linked to the scope.
    // Use for: DOM manipulation, initializing third-party plugins, scroll position.
    // Runs after $onInit.

    c.$postLink = function() {
        // Example: auto-focus a search input after widget renders
        var searchInput = document.querySelector('.sh-search-input');
        if (searchInput) searchInput.focus();
    };

    // ── $onChanges ─────────────────────────────────────────────────
    // Fires when a bound input property changes (one-way data binding).
    // Use for: reacting to parent widget passing new values.
    // Note: ScholarHub widgets are mostly standalone — this is less common.

    c.$onChanges = function(changes) {
        if (changes.scholarshipId && changes.scholarshipId.currentValue) {
            c.loadScholarshipDetail(changes.scholarshipId.currentValue);
        }
    };

    // ── $onDestroy ─────────────────────────────────────────────────
    // Fires when the widget is REMOVED from the page.
    // Use for: cleaning up intervals, timeouts, event listeners.
    // CRITICAL: Always clean up here to prevent memory leaks.

    c.$onDestroy = function() {
        // Cancel any pending $http requests
        if (c._loadCancel) c._loadCancel.resolve();

        // Clear any intervals
        if (c._pollingInterval) clearInterval(c._pollingInterval);

        // Remove any custom event listeners added with addEventListener
        // (Angular $scope.$on listeners clean up automatically)
    };
}
```

### Practical $onInit Patterns for ScholarHub

```javascript
// Widget: Application Review Detail
c.$onInit = function() {
    // Read the application sys_id from URL params
    var params = new URLSearchParams(window.location.search);
    c.applicationId = params.get('sys_id') || c.options.sys_id;

    if (!c.applicationId) {
        c.data.error = 'No application specified. Please return to the queue.';
        return;
    }

    // Server script already loaded the main data
    // Load the document list separately (keeps server script fast)
    c.loadDocuments();
};

c.loadDocuments = function() {
    c.docsLoading = true;
    $http.get('/api/now/table/sn_scholar_document?sysparm_query=application=' + c.applicationId)
        .then(function(response) {
            c.docsLoading = false;
            c.documents = response.data.result || [];
        })
        .catch(function() {
            c.docsLoading = false;
            c.docsError = 'Could not load documents.';
        });
};
```

---

## Part 3 — Angular Scope Events (Widget-to-Widget Communication)

### The Three Event Methods

```javascript
// $emit — sends event UP the scope chain (child → parent)
$scope.$emit('scholar.eligibility.checked', { status: 'eligible', scholarshipId: id });

// $broadcast — sends event DOWN the scope chain (parent → all children)
$scope.$broadcast('scholar.scholarship.selected', { sys_id: scholarshipId });

// $on — LISTENS for an event
$scope.$on('scholar.scholarship.selected', function(event, data) {
    c.selectedScholarshipId = data.sys_id;
    c.loadEligibilityForm();
});
```

### ScholarHub Widget Communication Patterns

```javascript
// ─── Pattern: Scholarship Detail page ──────────────────────────────
// Widget A (scholarship-detail) fires event when scholar selects scholarship
c.selectScholarship = function(scholarshipId) {
    c.selectedId = scholarshipId;
    $scope.$broadcast('scholar.scholarship.changed', {
        sys_id:          scholarshipId,
        name:            c.getScholarshipName(scholarshipId),
        type:            c.getScholarshipType(scholarshipId)
    });
};

// Widget B (ai-eligibility-check) listens and resets its form
$scope.$on('scholar.scholarship.changed', function(event, data) {
    c.scholarshipId   = data.sys_id;
    c.scholarshipType = data.type;
    c.aiResult        = null;  // clear previous result
    c.showForm        = true;
    // Dynamically show/hide income fields based on type
    c.isNeedBased = (data.type === 'need_based' || data.type === 'combined');
});
```

```javascript
// ─── Pattern: Reload widget after action completes ──────────────────
// After a UI Action (Approve button clicked), reload the review queue widget
c.approveApplication = function(applicationId) {
    c.actionInProgress = true;
    $http.post('/api/now/table/sn_scholar_application/' + applicationId, {
        data: { status: 'approved' }
    }).then(function() {
        c.actionInProgress = false;
        spUtil.addInfoMessage('Application approved successfully.');
        // Reload the queue widget above
        $scope.$emit('sp.widget.reload');
    }).catch(function() {
        c.actionInProgress = false;
        spUtil.addErrorMessage('Could not approve application. Please try again.');
    });
};
```

```javascript
// ─── Pattern: Cross-page notification after form submit ─────────────
// After submitting compliance, redirect with a success message
c.submitCompliance = function() {
    c.submitting = true;
    // ... submit logic ...
    .then(function() {
        // Store success message in sessionStorage (survives page redirect)
        sessionStorage.setItem('sh_flash_message', 'Compliance submitted successfully.');
        sessionStorage.setItem('sh_flash_type', 'success');
        window.location.href = '/scholarhub?id=my-awards';
    });
};

// On the My Awards page $onInit, check for flash message
c.$onInit = function() {
    var msg  = sessionStorage.getItem('sh_flash_message');
    var type = sessionStorage.getItem('sh_flash_type');
    if (msg) {
        if (type === 'success') spUtil.addInfoMessage(msg);
        else spUtil.addErrorMessage(msg);
        sessionStorage.removeItem('sh_flash_message');
        sessionStorage.removeItem('sh_flash_type');
    }
};
```

---

## Part 4 — In-Portal Notifications (spUtil Toasts)

### spUtil Message Reference

```javascript
// Available in the widget CLIENT CONTROLLER (not server script)

// Green success toast (auto-dismisses)
spUtil.addInfoMessage('Your application has been submitted.');

// Red error toast (persists until dismissed)
spUtil.addErrorMessage('Submission failed. Please check your documents.');

// Refresh the current widget
$scope.$emit('sp.widget.reload');

// Navigate to another page
window.location.href = '/scholarhub?id=my-apps';

// Navigate with params
window.location.href = '/scholarhub?id=scholarship&sys_id=' + scholarshipId;

// Open a modal with a widget inside
spUtil.openModal({
    widget: 'scholar-ai-check',
    widgetInput: { scholarship_id: scholarshipId }
});
```

### When to Use What

```
IN-PORTAL TOAST (spUtil.addInfoMessage / addErrorMessage):
  Use for: Immediate action feedback (saved, deleted, submitted, failed)
  Duration: Info = auto-dismiss ~3s | Error = stays until dismissed
  Example: After clicking "Approve" → "Application approved."

ALERT BANNER (sh-alert in widget HTML):
  Use for: Persistent page-level info (compliance due date, account on hold)
  Example: Banner at top of My Awards page when award is on hold

REDIRECT WITH FLASH (sessionStorage pattern):
  Use for: Success message that survives a page navigation
  Example: "Compliance submitted" after redirecting to My Awards

EMAIL NOTIFICATION:
  Use for: Events that happen while user is NOT on the portal
  Example: Application approved while applicant is offline
```

---

## Part 5 — Business Rule as Lifecycle Hooks

Think of Business Rules as lifecycle hooks on your data records:

```
Record lifecycle:         BR hook to use:
──────────────────────    ──────────────────────────────────────────────────
Before created            Before Insert  — validate, modify, or abort
After created             After Insert   — trigger flows, create related records
Before updated            Before Update  — validate state transitions, modify
After updated             After Update   — react to changes (status, field values)
Before deleted            Before Delete  — prevent deletion, archive data
After deleted             After Delete   — cleanup related records

Status change hook:
  current.status.changes()               — any status change
  current.status.changesTo('approved')   — specific destination
  current.status.changesFrom('draft')    — specific origin

Field change hook:
  current.field_name.changes()           — any change to this field
```

### ScholarHub Hook Patterns

```javascript
// Hook: "on scholarship deadline reached" — runs when auto-close BR fires
// Table: sn_scholar_scholarship | When: Before Update
// Condition: current.status.changesTo('closed')
(function executeRule(current, previous) {
    if (!current.status.changesTo('closed')) return;

    // Log the closing event
    gs.log('ScholarHub: Scholarship ' + current.getValue('number') + ' closed. ' +
           'Received: ' + getApplicationCount(current.getUniqueValue()) + ' applications.', 'ScholarHub');

    // Fire event for any post-close processing
    gs.eventQueue('x_snc_scholar.scholarship.closed', current,
                  current.getValue('provider'), current.getValue('number'));
})(current, previous);

// Helper
function getApplicationCount(scholarshipId) {
    var ga = new GlideAggregate('sn_scholar_application');
    ga.addQuery('scholarship', scholarshipId);
    ga.addAggregate('COUNT');
    ga.query();
    return ga.next() ? parseInt(ga.getAggregate('COUNT')) : 0;
}
```

```javascript
// Hook: "on award placed on hold" — reactive to compliance non-compliance
// Table: sn_scholar_award | When: After Update
// Condition: current.status.changesTo('on_hold')
(function executeRule(current, previous) {
    if (!current.status.changesTo('on_hold')) return;

    // Fire event (notification handles email)
    gs.eventQueue('x_snc_scholar.award.on_hold', current,
                  current.getValue('scholar'),
                  current.getDisplayValue('scholarship'));

    gs.log('ScholarHub: Award ' + current.getValue('number') + ' placed on hold.', 'ScholarHub');
})(current, previous);
```

---

## Quick Reference Card

```
NEED TO...                              USE...
────────────────────────────────────    ────────────────────────────────────────────
Trigger an email on a custom event      gs.eventQueue + Notification (Event fired)
Run code when an event fires            Script Action listening to event name
React to form field change              Client Script (onChange)
Validate before record saves            Business Rule (Before Insert/Update)
React after record saves                Business Rule (After Insert/Update)
Run code when widget loads              c.$onInit in client controller
React to page render complete           c.$postLink in client controller
Clean up timers/listeners               c.$onDestroy in client controller
Send message to another widget          $scope.$broadcast (parent→child)
Send message to parent widget           $scope.$emit (child→parent)
Listen for widget event                 $scope.$on('event.name', handler)
Show toast after action                 spUtil.addInfoMessage / addErrorMessage
Show error in portal widget             data.error + ng-if="c.data.error" in template
Pass message across page navigation     sessionStorage + $onInit flash check
Reload a widget after action            $scope.$emit('sp.widget.reload')
```
