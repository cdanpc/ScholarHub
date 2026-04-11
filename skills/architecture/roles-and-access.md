# Roles & Access Control — ScholarHub
> Agent: use this for every ACL, User Criteria, and role-check in scripts.

---

## The Three Roles

```
sn_scholar.applicant    Student / Applicant
sn_scholar.provider     Scholarship Provider (org, foundation, university)
sn_scholar.admin        System Administrator
```

**Checking roles in scripts:**
```javascript
// Server-side (Business Rule, Script Include, Flow Action)
gs.hasRole('sn_scholar.provider')    // true/false
gs.hasRole('sn_scholar.admin')
gs.getUserID()                        // current user sys_id
gs.getUserName()                      // current user login name

// Client-side (Client Script) — use GlideAjax, never check roles directly
// OR use: g_user.hasRole('sn_scholar.provider')  [available in portal]
```

---

## ACL Matrix — What Each Role Can Do

### sn_scholar_scholarship

| Operation | applicant | provider | admin |
|-----------|-----------|----------|-------|
| Read (status=open) | YES | YES | YES |
| Read (status=draft/closed) | NO | Own records only | YES |
| Create | NO | YES | YES |
| Update | NO | Own records only | YES |
| Delete | NO | NO | YES |

**ACL script for provider own-record access:**
```javascript
// Condition for provider read/write their own scholarships:
answer = gs.hasRole('sn_scholar.provider') &&
         current.provider == gs.getUserID();
```

### sn_scholar_application

| Operation | applicant | provider | admin |
|-----------|-----------|----------|-------|
| Read | Own only | Applications for their scholarships | ALL |
| Create | YES (own) | NO | YES |
| Update (status=draft) | Own only | NO | YES |
| Update (status field) | NO | Own scholarships | YES |
| Delete | NO | NO | YES |

**ACL script for applicant own-record:**
```javascript
answer = gs.hasRole('sn_scholar.applicant') &&
         current.applicant == gs.getUserID();
```

**ACL script for provider access to their scholarship's applications:**
```javascript
answer = gs.hasRole('sn_scholar.provider') &&
         current.scholarship.provider == gs.getUserID();
```

### sn_scholar_document

| Operation | applicant | provider | admin |
|-----------|-----------|----------|-------|
| Read | Own application's docs | Their scholarship's docs | ALL |
| Create/Update | Own application | NO | YES |
| Delete | NO | NO | YES |

### sn_scholar_award

| Operation | applicant | provider | admin |
|-----------|-----------|----------|-------|
| Read | Own only | Their scholarship's awards | ALL |
| Create | NO | YES (on approval) | YES |
| Update (disbursement_status) | NO | Their scholarships | YES |
| Delete | NO | NO | YES |

### sn_scholar_compliance

| Operation | applicant | provider | admin |
|-----------|-----------|----------|-------|
| Read | Own only | Their scholarship's scholars | ALL |
| Create | YES (own active award) | NO | YES |
| Update (status field) | NO | Their scholarship's scholars | YES |
| Delete | NO | NO | YES |

**ACL script — applicant can only create compliance if they have an active award:**
```javascript
if (operation == 'create') {
  var awardGR = new GlideRecord('sn_scholar_award');
  awardGR.addQuery('scholar', gs.getUserID());
  awardGR.addQuery('status', 'active');
  awardGR.query();
  answer = awardGR.hasNext();
} else {
  answer = current.award.scholar == gs.getUserID();
}
```

---

## Service Portal User Criteria

User Criteria controls which widgets/pages are visible. Applied at the widget level in the portal theme editor.

| Widget / Page | Criteria Rule | Script |
|---------------|---------------|--------|
| Scholarship Catalog | All authenticated users | `gs.isLoggedIn()` |
| AI Eligibility Check | applicant role | `gs.hasRole('sn_scholar.applicant')` |
| Submit Application form | applicant role | `gs.hasRole('sn_scholar.applicant')` |
| My Applications page | applicant role | `gs.hasRole('sn_scholar.applicant')` |
| My Awards page | applicant role + has award | see below |
| Compliance Submit | applicant + active award | see below |
| Post Scholarship form | provider role | `gs.hasRole('sn_scholar.provider')` |
| Application Queue | provider role | `gs.hasRole('sn_scholar.provider')` |
| Scholar Roster | provider role | `gs.hasRole('sn_scholar.provider')` |
| Compliance Review | provider role | `gs.hasRole('sn_scholar.provider')` |
| Admin tools | admin role | `gs.hasRole('sn_scholar.admin')` |

**User Criteria script — applicant with active award:**
```javascript
// Used for My Awards and Compliance Submit widgets
if (!gs.hasRole('sn_scholar.applicant')) return false;
var award = new GlideRecord('sn_scholar_award');
award.addQuery('scholar', gs.getUserID());
award.addQuery('status', 'active');
award.setLimit(1);
award.query();
return award.hasNext();
```

---

## Creating User Criteria in ServiceNow

1. Navigate to: **Service Portal → User Criteria**
2. Click **New**
3. Set Name (e.g., "ScholarHub Applicant Role")
4. Check "Script" checkbox
5. Enter the script in the Script field
6. Save
7. Apply to widget: open the widget → Options tab → User Criteria field

---

## How to Impersonate Users for Testing

In ServiceNow Studio / main instance:
1. Click your user avatar (top right)
2. Click **Impersonate User**
3. Search for test user (e.g., `applicant@scholarhub.test`)
4. Click **Impersonate**
5. Test the portal as that user
6. To stop: click avatar → **End Impersonation**

**Always test with impersonation — never assume your admin account reflects what the user sees.**

---

## Common Access Control Mistakes

- **Never** check roles in a Client Script directly (`g_form.getControl()` is unreliable for this). Use GlideAjax to call a Script Include that checks roles server-side.
- **Admin role does not automatically include applicant or provider** — they are separate. Test admin access separately.
- **User Criteria is evaluated at page load** — if a user's role changes mid-session, they must refresh the portal.
- **ACLs are evaluated in order** — if two ACLs match, both must pass. Design ACLs to be non-overlapping.
