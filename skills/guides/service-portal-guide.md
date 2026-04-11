# Service Portal Guide — ServiceNow
> Agent: the portal is what judges see first. Build widgets that are clean, role-correct, and error-handled.

---

## Widget Anatomy

Every Service Portal widget has 5 parts:

```
HTML Template      → The UI (Angular.js template syntax: {{data.field}}, ng-if, ng-repeat)
CSS / SCSS         → Styles (scoped to the widget)
Client Controller  → Angular.js $scope logic, $http calls, user interactions
Server Script      → GlideRecord queries, returns data object to client
Option Schema      → Configurable options (like widget title, record sys_id)
```

---

## Server Script → Client Data Flow

```javascript
// SERVER SCRIPT — runs on the server when widget loads
(function() {
    // GlideRecord queries go here
    var scholarships = [];
    var gr = new GlideRecord('sn_scholar_scholarship');
    gr.addQuery('status', 'open');
    gr.orderBy('application_deadline');
    gr.query();
    while (gr.next()) {
        scholarships.push({
            sys_id: gr.getUniqueValue(),
            name: gr.getDisplayValue('name'),
            type: gr.getValue('scholarship_type'),
            gwa_req: gr.getValue('gwa_requirement'),
            income_cap: gr.getDisplayValue('income_cap'),
            deadline: gr.getDisplayValue('application_deadline'),
            slots: gr.getValue('slots_remaining') || gr.getValue('slots_available')
        });
    }

    data.scholarships = scholarships;  // 'data' object is shared with client
    data.isApplicant = gs.hasRole('sn_scholar.applicant');
    data.isProvider = gs.hasRole('sn_scholar.provider');
    data.currentUserId = gs.getUserID();
})();
```

```javascript
// CLIENT CONTROLLER — Angular.js controller
function($scope, $http, spUtil) {
    // data from server is available as c.data (c = this controller)
    var c = this;
    // c.data.scholarships, c.data.isApplicant, etc. are already populated

    // Function triggered by user action
    c.applyNow = function(scholarshipId) {
        // Redirect to application form page
        window.location.href = '/scholarhub?id=apply&scholarship=' + scholarshipId;
    };

    c.checkEligibility = function(scholarshipId) {
        c.loading = true;
        $http.get('/api/now/table/sn_scholar_scholarship/' + scholarshipId)
            .then(function(response) {
                // handle response
                c.loading = false;
            }).catch(function(err) {
                c.loading = false;
                c.errorMsg = 'Could not load scholarship details.';
            });
    };
}
```

```html
<!-- HTML TEMPLATE -->
<div class="scholar-catalog">
  <div ng-if="c.data.scholarships.length == 0" class="empty-state">
    <p>No scholarships available at this time.</p>
  </div>

  <div class="scholarship-card" ng-repeat="s in c.data.scholarships">
    <h3>{{s.name}}</h3>
    <p><strong>Type:</strong> {{s.type | capitalize}}</p>
    <p ng-if="s.gwa_req"><strong>Min GWA:</strong> {{s.gwa_req}}</p>
    <p ng-if="s.income_cap"><strong>Max Income:</strong> {{s.income_cap}}</p>
    <p><strong>Deadline:</strong> {{s.deadline}}</p>
    <p><strong>Slots:</strong> {{s.slots}}</p>
    <button ng-if="c.data.isApplicant" ng-click="c.checkEligibility(s.sys_id)"
            class="btn btn-primary">Check Eligibility</button>
    <button ng-if="c.data.isApplicant" ng-click="c.applyNow(s.sys_id)"
            class="btn btn-success">Apply Now</button>
  </div>
</div>
```

---

## Angular.js Patterns in Service Portal

```html
<!-- Conditionals -->
<div ng-if="c.data.isApplicant">Only for applicants</div>
<div ng-show="c.loading">Loading...</div>
<div ng-hide="c.loading">Content</div>

<!-- Loops -->
<div ng-repeat="item in c.data.items track by item.sys_id">
  {{item.name}}
</div>

<!-- Click handlers -->
<button ng-click="c.myFunction()">Click me</button>
<button ng-click="c.myFunction(item.sys_id)">With param</button>

<!-- Form binding -->
<input ng-model="c.formData.gwa" type="number" />
<textarea ng-model="c.formData.essay"></textarea>

<!-- Class binding -->
<div ng-class="{'active': c.isSelected, 'error': c.hasError}">...</div>

<!-- Filter -->
<p>{{c.data.income | currency:'₱'}}</p>
<p>{{c.data.deadline | date:'MMM dd, yyyy'}}</p>
```

---

## ScholarHub Portal Structure

```
Portal ID: scholarhub
Portal URL: /scholarhub

Pages:
  homepage    (catalog) — default landing page
  scholarship — scholarship detail + AI pre-screen
  apply       — application form
  my-apps     — my applications tracker
  my-awards   — my awards
  compliance  — compliance submission
  my-schemas  — provider: my scholarships
  review      — provider: application review
  scholars    — provider: scholar roster

Page URL format:
  /scholarhub?id=[page-id]
  /scholarhub?id=scholarship&sys_id=[scholarship-sysid]
  /scholarhub?id=apply&scholarship=[scholarship-sysid]
  /scholarhub?id=review&sys_id=[application-sysid]
```

---

## Passing Parameters Between Pages

```javascript
// Navigate to page with parameters:
window.location.href = '/scholarhub?id=review&sys_id=' + applicationSysId;

// Read parameters in a widget server script:
var sysId = $sp.getParameter('sys_id');
var scholarshipId = $sp.getParameter('scholarship');

// Read parameters in client controller:
// Use spUtil or read from the URL directly
var params = new URLSearchParams(window.location.search);
var sysId = params.get('sys_id');
```

---

## spUtil Reference (Available in Client Controller)

```javascript
// Show loading overlay
spUtil.showLoadingSpinner();
spUtil.hideLoadingSpinner();

// Show notification toasts
spUtil.addInfoMessage('Operation successful');
spUtil.addErrorMessage('Something went wrong');

// Open a modal with a widget inside
spUtil.openModal({widget: 'widget-id', widgetInput: {sys_id: sysId}});

// Refresh a widget
$scope.$emit('sp.widget.reload');
```

---

## Portal Theming (ScholarHub Colors)

```css
/* In Portal CSS or Widget CSS */
:root {
  --scholar-teal: #0F6E56;
  --scholar-blue: #0C447C;
  --scholar-light: #E1F5EE;
  --scholar-text: #1A1A1A;
}

.scholar-card {
  border-left: 4px solid var(--scholar-teal);
  padding: 16px;
  margin-bottom: 12px;
  background: #fff;
  border-radius: 4px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}

.btn-scholar-primary {
  background-color: var(--scholar-teal);
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}
```

---

## Common Portal Mistakes

- **Forgetting empty state handling** — if `data.scholarships` is empty, `ng-repeat` shows nothing with no message. Always add an `ng-if` for the empty case.
- **Not using `track by`** in `ng-repeat` — causes Angular to re-render all items on any change. Always add `track by item.sys_id`.
- **Direct `window.location` vs `$location`** — in Service Portal, use `window.location.href` for navigation. `$location` service may not work as expected.
- **Server script runs on every widget load** — keep it fast. Don't run complex aggregations in the server script.
- **`data` object is not reactive** — changes to `data` after initial load won't update the HTML. Use `$scope` variables in the client controller for reactive state.
