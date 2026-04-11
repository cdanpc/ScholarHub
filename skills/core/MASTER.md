# MASTER INDEX — ScholarHub Agent Memory
> READ THIS FILE FIRST. Every session. Before any other file.
> This is your map. It tells you what to read, in what order, and what is already done.
> When a file's work is fully implemented and committed, mark it [x]. You will never read it again.

---

## HOW TO USE THIS FILE

```
STATUS MARKERS:
[ ]  Not yet read or implemented — load and act on this file
[~]  Currently in progress this session — resume here
[x]  Fully implemented, tested, and committed to GitHub — SKIP, do not re-read

RULE 1: Read this file before anything else. Every session.
RULE 2: Never load a file marked [x]. Its content is already built into the system.
RULE 3: Mark [~] on the file you are currently working on. One at a time.
RULE 4: Only mark [x] after: implemented + tested with impersonation + committed to GitHub + FEATURES.md updated.
RULE 5: Update this file at the end of every session (part of GATE 4 post-work).
```

---

## TIER 1 — ALWAYS LOAD (Every Session, Never Mark Done)

> These three files are loaded at the start of EVERY session without exception.
> Do not mark these [x]. They are permanent session context.

```
[ALWAYS]  FEATURES.md              Development tracker — read first to know current state
[ALWAYS]  session-gates.md         6 gates + 3 standing rules — governs how you work
[ALWAYS]  handoff-template.md      Session handoff schema — fill this at session end
```

**Session-start checklist (from session-gates.md GATE 1):**
```
1. Read FEATURES.md → know what is done, in-progress, blocked
2. Check OVERALL PROGRESS count
3. Read GATE STATUS table → see what gates ran last session
4. Note any BLOCKERS and DEAD ENDS
5. Then load Tier 2 files you have not yet marked [x]
6. Then load the Tier 3 file for today's work area
7. State your plan before writing code
```

---

## TIER 2 — LOAD ONCE (Project Context)

> Read each of these once at the start of the project.
> Mark [x] after you have fully absorbed them and they are reflected in your work.
> Do not re-read once marked [x] — the knowledge is already applied.

```
STATUS  FILE                        WHEN TO LOAD              WHAT IT GIVES YOU
──────  ──────────────────────────  ────────────────────────  ─────────────────────────────────────────
[ ]     project-brief.md            Session 1 (Day 1)         Project name, roles, stack, team, constraints
[ ]     data-model.md               Session 1 (Day 1)         All 5 tables, every field, GlideRecord queries
[ ]     roles-and-access.md         Session 1 (Day 1)         3 roles, ACL scripts, portal User Criteria
[ ]     process-flow.md             Session 1 (Day 1)         6 phases, status values, what triggers what
[ ]     mvp-scope.md                Session 1 (Day 1)         P0/P1/P2 features, out-of-scope, demo walkthrough
[ ]     naming-conventions.md       Session 1 (Day 1)         Every naming rule: tables, scripts, widgets, commits
[ ]     quality-contract.md         Session 1 (Day 1)         What "done" means per feature type
[ ]     danger-map.md               Session 1 (Day 1)         High-risk files and areas to handle carefully
[ ]     reference-credentials.md    Session 1 (Day 1)         Claude API key, GitHub PAT, test users, PDI URL
[ ]     reference-commands.md       Session 1 (Day 1)         Studio commands, git commands, ServiceNow URLs
[ ]     recurring-discipline.md     Session 1 (Day 1)         Daily, every-5-sessions, Day-6, Day-7 schedules
```

**Mark [x] rule for Tier 2:**
Mark a Tier 2 file [x] when you have read it AND confirmed its content is applied in your work
(e.g., you are using the correct naming conventions, you know the table field names, etc.).
You will not need to re-read it — its facts are part of your working knowledge for this project.

---

## TIER 3 — LOAD ON-DEMAND (Development Skills)

> Load only the file for what you are building right now.
> Mark [x] when the feature is: implemented + self-audited (GATE 3) + committed + FEATURES.md updated.
> Ordered by the 7-day sprint schedule.

### Day 1 — Foundation & Setup

```
STATUS  FILE                        COVERS
──────  ──────────────────────────  ─────────────────────────────────────────────────────────
[ ]     scoped-app-guide.md         Creating the Scoped App, Studio navigation, scope rules
[ ]     github-servicenow-workflow.md  Linking to GitHub, daily commit/push, branch strategy
```

### Day 2 — Backend Logic

```
STATUS  FILE                        COVERS
──────  ──────────────────────────  ─────────────────────────────────────────────────────────
[ ]     gliderecord-patterns.md     All server-side GlideRecord query, insert, update patterns
[ ]     business-rule-patterns.md   All 5 ScholarHub BRs with full scripts, trigger types
[ ]     acl-and-glideajax-patterns.md  ACL scripts for all 3 roles + GlideAjax server-client bridge
[ ]     validation-patterns.md      3-layer validation: client, server BR, flow — all ScholarHub rules
[ ]     error-handling-patterns.md  try/catch, GlideRecord failures, Flow faults, widget errors, logging
[ ]     events-and-hooks.md         gs.eventQueue, Script Actions, BR lifecycle hooks, Angular $onInit/$onDestroy
```

### Day 3 — Workflow & Notifications

```
STATUS  FILE                        COVERS
──────  ──────────────────────────  ─────────────────────────────────────────────────────────
[ ]     flow-designer-guide.md      Full lifecycle flow, wait conditions, subflows, debugging
[ ]     notification-guide.md       All 9 email templates + inbound email action script
```

### Day 3–4 — Service Portal

```
STATUS  FILE                        COVERS
──────  ──────────────────────────  ─────────────────────────────────────────────────────────
[ ]     service-portal-guide.md     Widget anatomy, Angular patterns, spUtil, page navigation
[ ]     service-portal-widget-catalog.md  All 11 widgets — data sources, role checks, page map
[ ]     ui-design-system.md         Color tokens, components (cards, buttons, badges, tables, forms)
[ ]     user-criteria-guide.md      5 User Criteria scripts, how to apply to widgets/pages
```

### Day 5 — AI, Integration & UI Layer

```
STATUS  FILE                        COVERS
──────  ──────────────────────────  ─────────────────────────────────────────────────────────
[ ]     integration-hub-guide.md    Step-by-step Claude API setup, REST Message, Flow Action
[ ]     claude-api-spec.md          Endpoint, payload, response parsing, error handling, caching
[ ]     ai-eligibility-logic.md     PH eligibility rules, prompt construction, portal display
[ ]     client-script-patterns.md   All 4 ScholarHub CSs, g_form API, GlideAjax callbacks
[ ]     ui-action-patterns.md       All 5 ScholarHub UAs with full scripts, role conditions
```

### Day 6 — Testing & Polish

```
STATUS  FILE                        COVERS
──────  ──────────────────────────  ─────────────────────────────────────────────────────────
[ ]     testing-checklist.md        12 test cases, impersonation steps, debug commands
```

### Day 7 — Delivery

```
STATUS  FILE                        COVERS
──────  ──────────────────────────  ─────────────────────────────────────────────────────────
[ ]     update-set-guide.md         Export Update Set, merge sets, XML submission for judges
```

---

## READING ORDER — First Session (Day 1, Clean Start)

If this is the very first session and nothing is marked, follow this exact sequence:

```
STEP 1  Read: MASTER.md (this file)         ← you are here
STEP 2  Read: FEATURES.md                   ← understand current build state
STEP 3  Read: session-gates.md              ← understand how to operate
STEP 4  Read: project-brief.md              ← understand what we're building
STEP 5  Read: data-model.md                 ← know all table names and fields
STEP 6  Read: roles-and-access.md           ← know the 3 roles and ACL patterns
STEP 7  Read: naming-conventions.md         ← lock in naming before writing anything
STEP 8  Read: quality-contract.md           ← know what "done" means
STEP 9  Read: danger-map.md                 ← know what to handle carefully
STEP 10 Read: reference-credentials.md     ← know where API keys and test users live
STEP 11 Read: reference-commands.md        ← know Studio and git commands
STEP 12 Read: scoped-app-guide.md          ← Day 1 task: create the Scoped App
STEP 13 Read: github-servicenow-workflow.md← Day 1 task: link to GitHub
STEP 14 State your Day 1 plan and begin
```

---

## READING ORDER — Subsequent Sessions

```
STEP 1  Read: MASTER.md (this file)
        → Scan for [~] entries (in-progress from last session)
        → Scan for [ ] entries in Tier 3 for today's scheduled work
        → Note all [x] entries — do not load those files

STEP 2  Read: FEATURES.md
        → Check OVERALL PROGRESS
        → Note what's done, in-progress, blocked
        → Identify today's focus tasks

STEP 3  Read: session-gates.md (the standing rules section only)
        → Remind yourself of the 3 standing rules

STEP 4  Load the Tier 3 file for today's work area (one file, the active one)
        → If Day 2: load gliderecord-patterns.md and business-rule-patterns.md
        → If Day 3: load flow-designer-guide.md and notification-guide.md
        → etc.

STEP 5  State your plan. Begin work.
```

---

## HOW TO MARK FILES

When a file's corresponding work is complete:

```
1. Find the file's row in this MASTER.md
2. Change [ ] to [x]
3. Verify checklist:
   - Feature implemented in ServiceNow?           ✓ / ✗
   - Self-audited (GATE 3 re-audit passed)?        ✓ / ✗
   - Tested with correct role impersonation?       ✓ / ✗
   - Committed and pushed to GitHub?               ✓ / ✗
   - FEATURES.md tasks marked [x]?                ✓ / ✗
   All 5 must be ✓ before marking [x] here.
4. Save MASTER.md
5. Commit: "docs: mark [x] [filename] — fully implemented"
```

---

## QUICK LOOKUP — File by Task

When you need to do something specific, find the right file here:

```
TASK                                LOAD THIS FILE
─────────────────────────────────   ──────────────────────────────────
Create a table or add a field       data-model.md
Write a GlideRecord query           gliderecord-patterns.md
Write a Business Rule               business-rule-patterns.md
Validate a form or record           validation-patterns.md
Handle errors in any script         error-handling-patterns.md
Fire or listen for an event         events-and-hooks.md
Use Angular lifecycle hooks         events-and-hooks.md
Show in-portal toast notification   events-and-hooks.md (Part 4)
Write a Client Script               client-script-patterns.md
Call server from a Client Script    acl-and-glideajax-patterns.md
Create a UI Action                  ui-action-patterns.md
Build a Flow Designer step          flow-designer-guide.md
Configure email notifications       notification-guide.md
Build a Service Portal widget       service-portal-guide.md + ui-design-system.md
Know which widgets to build         service-portal-widget-catalog.md
Apply User Criteria to a widget     user-criteria-guide.md
Set up Claude API integration       integration-hub-guide.md + claude-api-spec.md
Build the AI eligibility widget     ai-eligibility-logic.md
Configure ACLs for a table          acl-and-glideajax-patterns.md
Set up GitHub source control        github-servicenow-workflow.md
Run the 12 test cases               testing-checklist.md
Export Update Set for submission    update-set-guide.md
Check naming for anything           naming-conventions.md
Understand what "done" means        quality-contract.md
Know what not to touch carelessly   danger-map.md
Remember API keys / test users      reference-credentials.md
Run Studio or git commands          reference-commands.md
Know the 7-day schedule             FEATURES.md (Phase headers)
End the session cleanly             handoff-template.md
```

---

## CURRENT SESSION FOCUS

> Agent: fill this in at the start of each session. Clear it at session end.

```
Session date:       [fill in]
Member working:     [Member 1 / 2 / 3 / 4 / 5]
Sprint day:         [Day 1–7]
Files loaded today: [list the files read this session]
Current task:       [one sentence — what are you building right now]
Blocking issue:     [none / or describe]
```

---

## MASTER INDEX CHANGE LOG

> Agent: append one line per session when you update this file.

```
[DATE]  Session 1  — Initial setup. No files marked [x] yet.
```

---

_This file is maintained by the agent. Commit every time a status marker changes._
_File path: /skills/MASTER.md — Read before any other file, every session._
