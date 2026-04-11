# ScholarHub Agent Skills

> Load only the subfolder(s) relevant to your current task to keep Claude's context focused and efficient.

---

## 📁 Folder Structure

```
skills/
├── core/           → Start here. Project foundation, scope, and AI logic.
├── patterns/       → Code-level implementation patterns (scripts, rules, queries).
├── guides/         → Step-by-step how-to guides for specific ServiceNow modules.
├── architecture/   → System design, data model, access control, and UI standards.
├── workflow/       → Process flows, handoffs, DevOps, and team discipline.
└── reference/      → Quick lookups: commands, credentials, events, API spec.
```

---

## 🗂️ core/
> **Load when:** Starting a new session, planning features, or working on AI/eligibility logic.

| File | Purpose |
|---|---|
| `MASTER.md` | Single source of truth — links to all other skills |
| `FEATURES.md` | Full feature list and implementation status |
| `project-brief.md` | Project goals, stakeholders, and constraints |
| `mvp-scope.md` | MVP boundaries and what's out of scope |
| `ai-eligibility-logic.md` | AI decision logic for scholarship eligibility |
| `testing-checklist.md` | QA checklist for all major features |

---

## 🗂️ patterns/
> **Load when:** Writing or reviewing GlideRecord queries, business rules, client scripts, ACLs, or validation logic.

| File | Purpose |
|---|---|
| `gliderecord-patterns.md` | GlideRecord query and update patterns |
| `business-rule-patterns.md` | Business rule triggers, conditions, and scripts |
| `client-script-patterns.md` | Client-side scripting patterns and best practices |
| `ui-action-patterns.md` | UI action button patterns and handlers |
| `acl-and-glideajax-patterns.md` | ACL rules and GlideAjax communication patterns |
| `error-handling-patterns.md` | Error handling and logging strategies |
| `validation-patterns.md` | Field and form validation approaches |

---

## 🗂️ guides/
> **Load when:** Building or debugging a specific ServiceNow module (portal, flows, integrations, notifications).

| File | Purpose |
|---|---|
| `service-portal-guide.md` | Service Portal setup and configuration |
| `service-portal-widget-catalog.md` | Widget catalog and reuse patterns |
| `flow-designer-guide.md` | Flow Designer usage and best practices |
| `integration-hub-guide.md` | Integration Hub spokes and connections |
| `notification-guide.md` | Email/push notification setup |
| `update-set-guide.md` | Update set management and migration |
| `scoped-app-guide.md` | Scoped application structure and rules |
| `user-criteria-guide.md` | User criteria configuration guide |

---

## 🗂️ architecture/
> **Load when:** Designing tables, defining roles, reviewing UI standards, or checking access control.

| File | Purpose |
|---|---|
| `data-model.md` | Table structure, fields, and relationships |
| `roles-and-access.md` | Role definitions and permission matrix |
| `session-gates.md` | Session-based access control logic |
| `naming-conventions.md` | Naming rules for tables, fields, and scripts |
| `ui-design-system.md` | UI components, colors, typography, and layout |
| `danger-map.md` | Known gotchas, anti-patterns, and risky areas |

---

## 🗂️ workflow/
> **Load when:** Coordinating handoffs, managing DevOps pipelines, or enforcing team process.

| File | Purpose |
|---|---|
| `process-flow.md` | End-to-end process and state machine |
| `github-servicenow-workflow.md` | GitHub ↔ ServiceNow DevOps workflow |
| `handoff-template.md` | Standard handoff format between agents/devs |
| `quality-contract.md` | Definition of done and quality standards |
| `recurring-discipline.md` | Recurring tasks and maintenance schedule |

---

## 🗂️ reference/
> **Load when:** You need a quick lookup — commands, credentials, event names, or the Claude API spec.

| File | Purpose |
|---|---|
| `reference-commands.md` | Common CLI and admin commands |
| `reference-credentials.md` | Credential locations and naming (no secrets stored here) |
| `events-and-hooks.md` | ServiceNow event registry and hook catalog |
| `claude-api-spec.md` | Claude API usage spec for ScholarHub integrations |

---

## ⚡ Recommended Loading Patterns

| Task | Load These Folders |
|---|---|
| Starting a new session | `core/` |
| Writing a business rule | `patterns/` + `architecture/` |
| Building a portal widget | `guides/` + `patterns/` + `architecture/` |
| Debugging an integration | `guides/` + `reference/` |
| Handing off to another agent | `workflow/` + `core/` |
| Quick lookup / sanity check | `reference/` |
