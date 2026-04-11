# Scoped Application Guide — ServiceNow
> Agent: ScholarHub lives in a Scoped Application. These rules govern everything we build.

---

## Scoped Application Basics

A Scoped Application is an isolated package in ServiceNow with its own namespace. Everything ScholarHub creates — tables, scripts, flows, notifications — lives inside this scope.

```
Application Name:  ScholarHub
Scope:             x_snc_scholar  (or team-assigned prefix)
Version:           1.0.0
Tables prefix:     sn_scholar_
Script prefix:     sn_scholar_  (for Business Rules, Script Includes)
```

---

## Creating the Application (Day 1)

```
1. Open ServiceNow Studio (navigate to /studio.do)
2. Click "Create Application"
3. Fill in:
   Name: ScholarHub
   Scope: x_snc_scholar (auto-generated or enter manually)
   Description: Scholarship Lifecycle Management System
4. Click "Create"
5. Studio opens with your new empty application
```

---

## Studio Navigation

```
Studio sidebar:
├── Data Model
│   └── Tables         ← Create all 5 sn_scholar_ tables here
├── Server Development
│   ├── Business Rules ← All 5 BRs go here
│   └── Script Includes← ScholarUtils, AIEligibilityHelper
├── Client Development
│   ├── Client Scripts ← All 4 CSs go here
│   └── UI Actions     ← All 5 UAs go here
├── Service Portal
│   ├── Portals        ← scholarhub portal
│   ├── Pages          ← All portal pages
│   └── Widgets        ← All portal widgets
├── Workflow
│   └── Flow Designer  ← Application Lifecycle Flow
└── Email
    ├── Notifications  ← All 9 outbound templates
    └── Inbound Actions← Inbound email approval
```

---

## Creating Tables in Studio

```
Studio → Data Model → Tables → New
Fill in:
  Label: Scholarship (display name)
  Name: sn_scholar_scholarship (auto-populated from label — verify it matches)
  Extends table: task (for workflow support) OR leave blank for simple tables
  Add module to menu: YES (for admin access)
  Create access controls: YES

After creating, add fields:
  Studio → table → Add Field
  Set: Name, Type, Required, Default value
```

**Which tables extend [task]?**
- `sn_scholar_application` — extends task (enables Flow Designer trigger on it)
- `sn_scholar_compliance` — extends task (optional but useful for workflow)
- Others can be standalone (not extending task)

---

## Application Scope Gotchas

- **Cross-scope access**: By default, your scoped app can only directly read/write tables in your scope. To read `sys_user` (global scope), use `GlideRecord` normally — global tables are accessible from any scope.
- **Script Includes** in your scope are only callable from your scope unless you set them as "Accessible from: All application scopes"
- **Notifications**: Create notifications within the scoped app — they automatically associate with your tables
- **Flow Designer**: Flows created in Studio are scoped to your application automatically

---

## Linking to GitHub (Day 1)

```
Studio → Source Control → Link to Repository

Fill in:
  URL: https://github.com/[team-org]/scholarhub-servicenow.git
  Branch: main
  Credential: [GitHub credential record with PAT]

Click "Link to Repository"
```

**Create the GitHub credential first:**
```
Navigate to: /sys_auth_profile_basic_list.do
New:
  Name: GitHub PAT - ScholarHub
  Type: Basic
  User name: [your GitHub username]
  Password: [your PAT token]
```
