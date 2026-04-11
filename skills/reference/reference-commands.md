# Reference Commands — ScholarHub
> Agent: use these exact commands. Don't guess at syntax — it wastes sessions.

---

## ServiceNow Studio — Source Control

```
Link repository (Day 1, one time only):
  Studio → Source Control → Link to Repository
  URL: https://github.com/[team-org]/scholarhub-servicenow.git
  Branch: main
  Credential: GitHub PAT (stored as ServiceNow Basic Auth credential)

Commit and push changes:
  Studio → Source Control → Commit Changes
  → Select changed files → Enter commit message → Commit
  → Studio → Source Control → Push Changes

Pull latest from GitHub:
  Studio → Source Control → Apply Remote Changes

Check current status:
  Studio → Source Control → Show Changes

Create a branch:
  Studio → Source Control → Create Branch
  Name format: feature/[feature-name] or fix/[bug-name]

Switch branch:
  Studio → Source Control → Switch Branch
```

---

## ServiceNow Navigation (URL Patterns)

```
Main instance:         https://[instance-name].service-now.com
Studio:                https://[instance-name].service-now.com/studio.do
Service Portal:        https://[instance-name].service-now.com/scholarhub
System Log:            https://[instance-name].service-now.com/syslog_list.do
Flow Designer:         https://[instance-name].service-now.com/flow-designer
Integration Hub:       https://[instance-name].service-now.com/x_snc_ihe_spoke_registry_list.do
User Criteria:         https://[instance-name].service-now.com/sp_user_criteria_list.do
ACL Rules:             https://[instance-name].service-now.com/sys_security_acl_list.do
Business Rules:        https://[instance-name].service-now.com/sys_script_list.do
Client Scripts:        https://[instance-name].service-now.com/sys_script_client_list.do
Notifications:         https://[instance-name].service-now.com/sysevent_email_action_list.do
Scheduled Jobs:        https://[instance-name].service-now.com/sysauto_list.do
```

---

## Testing & Debugging Commands

```
Run a background script (test GlideRecord queries):
  All Applications > System Definition > Scripts - Background
  Or: Navigate to /scripts_background.do

Impersonate a user:
  Click your avatar (top right) → Impersonate User → search test user → Impersonate

Stop impersonation:
  Click your avatar → End Impersonation

Check System Log (most recent errors):
  Navigate to: /syslog_list.do?sysparm_query=level=1^ORlevel=2&sysparm_view=

Manually trigger a scheduled Business Rule:
  Navigate to: /sysauto_list.do
  Find the scheduled job → Right-click → Execute Now

Test a REST endpoint (Integration Hub):
  Navigate to the REST Message → HTTP Methods tab → Test

Flow Designer execution history:
  Flow Designer → click flow → Execution Details tab

Clear a widget server script cache (if widget not updating):
  Add ?sysparm_no_cache=true to the portal URL
```

---

## Git Commands (From Terminal / VS Code)

```bash
# After cloning from GitHub
git clone https://github.com/[team-org]/scholarhub-servicenow.git
cd scholarhub-servicenow
git checkout -b feature/my-feature

# Before starting work
git pull origin main

# Check what changed
git status
git diff

# Commit work
git add .
git commit -m "feat: add validate_documents business rule"
git push origin feature/my-feature

# Check recent commits
git log --oneline -10

# See who changed what
git log --oneline --all --graph

# Tag the hackathon release (Day 7)
git tag v1.0-hackathon
git push origin v1.0-hackathon
```

---

## Commit Message Format

```
feat: [description]     → new feature
fix: [description]      → bug fix
test: [description]     → adding tests
docs: [description]     → documentation only
refactor: [description] → code change, no behavior change
style: [description]    → formatting, naming

Examples:
feat: add sn_scholar_validate_documents business rule
feat: implement Flow Designer wait condition for approval
fix: correct GWA indicator client script onChange trigger
feat: add AI eligibility pre-screen widget to scholarship detail page
docs: update FEATURES.md phase 3 progress
```

---

## Export Update Set (Day 7)

```
1. Navigate to: All Applications > System Update Sets > Local Update Sets
2. Find update set named "ScholarHub Development" (or create one if missing)
3. Click the update set name
4. Click "Export to XML"
5. Save as: ScholarHub_v1.0_[YYYYMMDD].xml
6. Share with team (upload to GitHub or shared drive)
```
