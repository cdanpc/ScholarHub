# User Criteria Guide — ServiceNow Service Portal
> Agent: User Criteria controls which authenticated users see which portal pages and widgets.

---

## What User Criteria Does

User Criteria is a set of rules attached to Service Portal widgets, catalog items, and knowledge articles. When a user loads a portal page, ServiceNow evaluates whether the user meets the criteria. If not — the widget is hidden entirely (not just disabled).

---

## Creating User Criteria Records

```
Navigate to: Service Portal → User Criteria → New
OR: sp_user_criteria_list.do → New

Fields:
  Name:          [descriptive, e.g., "ScholarHub Applicant Role"]
  Match all:     true = AND logic, false = OR logic
  Roles:         (quick option) select role directly
  Script:        (advanced) write custom JavaScript
  Active:        YES
```

---

## ScholarHub User Criteria Scripts

### UC1: sn_scholar.applicant role only
```javascript
// Name: ScholarHub - Applicant Role
answer = gs.hasRole('sn_scholar.applicant');
```

### UC2: sn_scholar.provider role only
```javascript
// Name: ScholarHub - Provider Role
answer = gs.hasRole('sn_scholar.provider');
```

### UC3: sn_scholar.admin role only
```javascript
// Name: ScholarHub - Admin Role
answer = gs.hasRole('sn_scholar.admin');
```

### UC4: Applicant with active award (for compliance page)
```javascript
// Name: ScholarHub - Applicant with Active Award
if (!gs.hasRole('sn_scholar.applicant')) {
    answer = false;
    return;
}
var award = new GlideRecord('sn_scholar_award');
award.addQuery('scholar', gs.getUserID());
award.addQuery('status', 'active');
award.setLimit(1);
award.query();
answer = award.hasNext();
```

### UC5: All authenticated users (catalog browsing)
```javascript
// Name: ScholarHub - Any Logged In User
answer = gs.isLoggedIn();
```

---

## Applying User Criteria to a Widget

```
Method A — In the Page Designer:
  1. Open Service Portal Designer (/sp_config)
  2. Navigate to your portal page
  3. Click on the widget you want to restrict
  4. In the right panel, find "User Criteria"
  5. Add the relevant criteria record

Method B — In the Widget Record:
  1. Navigate to the widget record directly
  2. Look for the User Criteria related list at the bottom
  3. Add a new row linking to the criteria record
```

---

## Applying User Criteria to Portal Pages

```
Navigate to: Service Portal → Pages → [your page]
User Criteria field: add the criteria record

This hides the ENTIRE PAGE from users who don't match.
Use this for high-level access (e.g., entire provider section).
Use widget-level criteria for fine-grained control within a page.
```

---

## Testing User Criteria

```
1. Create/verify the User Criteria record is Active
2. Apply it to your widget or page
3. Impersonate applicant@scholarhub.test
4. Navigate to the portal page
5. Verify: the widget IS visible (applicant should see catalog, form, etc.)
6. Impersonate provider@scholarhub.test
7. Navigate to the same page
8. Verify: the applicant-only widget is HIDDEN

Always test both positive (should see) and negative (should NOT see) cases.
```

---

## Common Mistakes

- **User Criteria is evaluated at page load** — role changes mid-session need a page refresh
- **Hidden widgets still run their server scripts** — this is a known ServiceNow behavior. For sensitive data, also add role checks inside the widget server script.
- **`answer` variable must be set** — if the script errors without setting `answer`, access is denied by default
- **Don't use `current` in User Criteria** — there is no current record context here, only session info
