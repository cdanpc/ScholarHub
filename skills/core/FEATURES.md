# ScholarHub — Development Tracker
> AI-maintained. Read at every session start. Update statuses after every work block.
> Platform: ServiceNow Scoped Application | Team: 5 members | Sprint: 7 days
> Repo: scholarhub-servicenow | Instance: [SN_INSTANCE_URL]

## STATUS KEY
```
[ ] not started   [~] in progress   [x] done   [!] blocked   [?] needs review
```

---

## OVERALL PROGRESS
<!-- Agent: update these counts after each session -->
- **Total tasks:** 103
- **Done:** 0 (0%)
- **In progress:** 0
- **Blocked:** 0
- **Remaining:** 103
- **Last updated:** not started
- **Active member today:** -

---

## SESSION LOG
<!-- Agent: append one row per session -->
| # | Date | Member | Summary | Gate Status |
|---|------|--------|---------|-------------|
| - | - | - | Not started | - |

---

## PHASE 1 — Foundation & Setup
> Owner: All members | Target: Day 1 (Monday)

### Scoped Application
- [ ] Create Scoped Application in ServiceNow Studio (`sn_scholar`)
- [ ] Set application scope: `x_snc_scholar` (or team-assigned prefix)
- [ ] Configure application manifest (version 1.0.0, vendor: ScholarHub Team)
- [ ] Create GitHub repository: `scholarhub-servicenow`
- [ ] Create GitHub PAT with scopes: `repo`, `workflow`
- [ ] Link ServiceNow Studio → Source Control → Link to Repository
- [ ] Test first commit and push from Studio
- [ ] Confirm repo structure is clean (no credentials, no .env)

### Tables (Member 1)
- [ ] `sn_scholar_scholarship` — 12 fields (see data-model.md)
- [ ] `sn_scholar_application` — 14 fields (see data-model.md)
- [ ] `sn_scholar_document` — 5 fields (see data-model.md)
- [ ] `sn_scholar_award` — 11 fields (see data-model.md)
- [ ] `sn_scholar_compliance` — 11 fields (see data-model.md)
- [ ] All reference field relationships configured
- [ ] Auto-number fields: SCH#, APP#, AWD# prefixes

### Roles & ACLs (Member 1)
- [ ] Role: `sn_scholar.applicant` created
- [ ] Role: `sn_scholar.provider` created
- [ ] Role: `sn_scholar.admin` created
- [ ] ACL: `sn_scholar_scholarship` — read(applicant+provider), CRUD(provider own, admin all)
- [ ] ACL: `sn_scholar_application` — CRU own(applicant), R+Update(provider), CRUD(admin)
- [ ] ACL: `sn_scholar_document` — CRU own(applicant), R(provider), CRUD(admin)
- [ ] ACL: `sn_scholar_award` — R own(applicant), CRU(provider), CRUD(admin)
- [ ] ACL: `sn_scholar_compliance` — CRU own(applicant with active award), CRU(provider), CRUD(admin)

### Test Users & Seed Data
- [ ] Test user: `applicant@scholarhub.test` → role: sn_scholar.applicant
- [ ] Test user: `provider@scholarhub.test` → role: sn_scholar.provider
- [ ] Test user: `admin@scholarhub.test` → role: sn_scholar.admin
- [ ] Seed: 3 sample scholarships (1 merit, 1 need-based, 1 combined)
- [ ] Seed: Sample applicant profile (GWA=90, income=250000, course=BSIT)

---

## PHASE 2 — Backend Logic
> Owner: Member 1 | Target: Days 1–2

### Business Rules
- [ ] BR: `sn_scholar_validate_documents` (Before Insert, sn_scholar_application)
  - Checks all required docs present; aborts + shows error list if missing
- [ ] BR: `sn_scholar_set_status_on_submit` (After Insert, sn_scholar_application)
  - Sets status=submitted, stamps submission_date, triggers Flow Designer
- [ ] BR: `sn_scholar_prevent_duplicate` (Before Insert, sn_scholar_application)
  - Blocks if active application for same scholarship already exists
- [ ] BR: `sn_scholar_flag_non_compliant` (Before Update, sn_scholar_compliance)
  - If status→non_compliant: set award.status=on_hold, queue notification
- [ ] BR: `sn_scholar_auto_close_scholarships` (Scheduled daily)
  - Sets status=closed for scholarships past deadline
- [ ] Test BR validate_documents: submit with missing PSA → blocked ✓
- [ ] Test BR set_status: submit complete app → status=submitted, date stamped ✓
- [ ] Test BR prevent_duplicate: submit second app for same scholarship → blocked ✓

### Script Includes
- [ ] `ScholarUtils` — helpers: `getRequiredDocs(scholarshipId)`, `checkDuplicate(userId, scholarshipId)`
- [ ] `AIEligibilityHelper` — `buildPrompt(applicationGR, scholarshipGR)`, `parseResponse(jsonString)`

---

## PHASE 3 — Workflow & Notifications
> Owner: Member 2 | Target: Day 3

### Flow Designer — Application Lifecycle Flow
- [ ] Create Flow: `ScholarHub Application Lifecycle`
- [ ] Trigger: Record Created — sn_scholar_application, status=submitted
- [ ] Step 1: Send outbound email to provider (New Application Received)
- [ ] Step 2: Set application status = under_review
- [ ] Step 3: Wait for condition (approval_action field set OR inbound email parsed)
- [ ] Branch A (Request More Docs): set status=draft, send Missing Docs email to applicant
  - Wait for resubmission → loop back to Step 1
- [ ] Branch B (Deny): capture denial_reason, set status=denied, send Denied email to applicant
- [ ] Branch C (Approve): set status=approved, trigger Award Creation subflow
- [ ] Subflow: `Create Award Record` — auto-creates sn_scholar_award from application data
- [ ] Step 4 (post-approve): Send Congratulations email to applicant
- [ ] Scheduled action: Compliance Due reminder (end of semester trigger)
- [ ] Test full approve path with test users ✓

### Email Templates (Outbound — 9 total)
- [ ] T1: `New Application Received` → provider
- [ ] T2: `Missing Documents Alert` → applicant (dynamic doc list)
- [ ] T3: `Application Under Review` → applicant
- [ ] T4: `Request for More Documents` → applicant (specific list from provider)
- [ ] T5: `Application Approved` → applicant (scholarship name, award details)
- [ ] T6: `Application Denied` → applicant (reason from provider)
- [ ] T7: `Award Disbursed` → scholar
- [ ] T8: `Compliance Submission Due` → scholar (semester, deadline)
- [ ] T9: `Non-Compliance Flagged` → scholar + provider
- [ ] All templates tested with test email addresses ✓

### Inbound Email Action
- [ ] Configure reply-to address for approval emails
- [ ] Create Inbound Email Action: `Process Approval Reply`
  - Parses `APPROVE` or `DENY [reason]` from email body
  - Maps to application record via email subject ref number
  - Triggers correct Flow Designer branch
- [ ] Test: reply APPROVE → application approved ✓
- [ ] Test: reply DENY not qualified → application denied with reason ✓

---

## PHASE 4 — Service Portal
> Owner: Member 3 | Target: Days 3–4

### Portal Foundation
- [ ] Create Service Portal: ID=`scholarhub`, title=ScholarHub
- [ ] Configure portal URL suffix: `/scholarhub`
- [ ] Apply theme: primary=#0F6E56 (teal), secondary=#0C447C (blue)
- [ ] Create main homepage layout
- [ ] Configure header with ScholarHub branding
- [ ] Configure role-based navigation (student nav vs provider nav)

### Student View — Pages & Widgets
- [ ] Page: `catalog` — Scholarship Catalog
  - [ ] Widget: Scholarship list cards (name, type, GWA req, income cap, deadline, slots)
  - [ ] Widget: Search bar + filter (by type, GWA, course, status=open only)
- [ ] Page: `scholarship` — Scholarship Detail
  - [ ] Widget: Full scholarship detail (all fields + benefits + required docs)
  - [ ] Widget: AI Eligibility Pre-screen (placeholder for Phase 5)
  - [ ] Widget: Apply Now button (visible to applicants only)
- [ ] Page: `apply` — Application Form
  - [ ] Widget: Dynamic application form (fields + document upload checklist)
  - [ ] Submit button → triggers Business Rule + Flow
- [ ] Page: `my-applications` — Application Tracker
  - [ ] Widget: Applications list with status timeline (submitted/under_review/approved/denied)
- [ ] Page: `my-awards` — Awards Page
  - [ ] Widget: Active awards (coverage, stipend, period, disbursement status)
- [ ] Page: `compliance` — Compliance Submit
  - [ ] Widget: Compliance upload form (grades + enrollment cert + semester info)
- [ ] User Criteria applied to all student widgets: `sn_scholar.applicant` role required
- [ ] Full student flow tested with applicant_test user ✓

### Provider View — Pages & Widgets
- [ ] Page: `my-scholarships` — Scholarship Management
  - [ ] Widget: My Scholarships list (name, slots remaining, status, applicant count)
  - [ ] Widget: Post New Scholarship form
  - [ ] Widget: Edit Scholarship form
- [ ] Page: `applications` — Application Review Queue
  - [ ] Widget: Application queue (sortable by scholarship, date, status)
- [ ] Page: `review` — Application Detail Review
  - [ ] Widget: Full applicant profile (all fields, AI pre-screen result)
  - [ ] Widget: Document list with view/download links
  - [ ] Widget: Approve / Deny / Request Docs action buttons
  - [ ] Widget: Review notes input
- [ ] Page: `scholars` — Scholar Roster
  - [ ] Widget: Active scholars (name, scholarship, GWA, compliance status)
- [ ] Page: `compliance-review` — Compliance Review
  - [ ] Widget: Compliance submissions (per-scholar, per-semester, with approve/flag actions)
- [ ] User Criteria applied to all provider widgets: `sn_scholar.provider` role required
- [ ] Full provider flow tested with provider_test user ✓

---

## PHASE 5 — AI, Client Scripts & UI Actions
> Owner: Member 4 (AI + Integration) + Member 5 (CS + UI Actions) | Target: Day 5

### Integration Hub — Claude API (Member 4)
- [ ] Create Connection Alias: `Anthropic Claude API`
- [ ] Store API key as ServiceNow Basic Auth Credential (header: x-api-key)
- [ ] Create REST Message: `claude_messages`
- [ ] Configure endpoint: `https://api.anthropic.com/v1/messages`
- [ ] Configure headers: `Content-Type: application/json`, `anthropic-version: 2023-06-01`
- [ ] Create REST Method: `check_eligibility` (POST)
- [ ] Build request body from AIEligibilityHelper.buildPrompt()
- [ ] Create Flow Designer Action: `AI Eligibility Pre-screen`
  - Input: gwa, family_income, course, school_type, honor_rank, scholarship_criteria_json
  - Output: eligibility_status (eligible/partial/not_eligible), matched_list, missing_list, note
- [ ] Implement error handling (timeout → show fallback message, parse error → log)
- [ ] Test call: eligible applicant (GWA 92, income 200k, IT course) ✓
- [ ] Test call: partial match (GWA meets, income over cap) ✓
- [ ] Test call: not eligible ✓
- [ ] Test API failure fallback ✓

### AI Eligibility Widget (Member 4)
- [ ] Widget: `ai-eligibility-check`
  - [ ] HTML template: result card (green/yellow/red status badge)
  - [ ] Show matched criteria as green checkmarks
  - [ ] Show missing requirements as red warnings
  - [ ] Show loading spinner during API call
  - [ ] Show fallback message on error
  - [ ] Angular controller: calls REST endpoint, updates scope on response
- [ ] Widget added to Scholarship Detail page ✓
- [ ] Result stored on application record (ai_pre_screen_result field) ✓

### Client Scripts (Member 4)
- [ ] CS: `Show Income Fields for Need-Based`
  - Table: sn_scholar_application | Type: onChange | Field: scholarship
  - Show/hide: family_income, itx_return doc field, indigency_cert, house_photo IF type=need_based OR combined
- [ ] CS: `Dynamic Document Checklist`
  - Table: sn_scholar_application | Type: onLoad
  - Fetch scholarship.required_documents via GlideAjax, render checkboxes dynamically
- [ ] CS: `GWA Eligibility Indicator`
  - Table: sn_scholar_application | Type: onChange | Field: gwa
  - Compare gwa vs scholarship.gwa_requirement; show green (meets) or amber (below) inline
- [ ] CS: `Lock Form After Decision`
  - Table: sn_scholar_application | Type: onLoad
  - If status IN (approved, denied): setReadOnly all fields, hide Submit button, show status banner
- [ ] All 4 Client Scripts tested in application form ✓

### UI Actions (Member 5)
- [ ] UA: `Submit Application`
  - Table: sn_scholar_application | Condition: status=draft | Role: sn_scholar.applicant
  - Client: confirm dialog | Server: validates form, sets status=submitted, triggers flow
- [ ] UA: `Approve Application`
  - Table: sn_scholar_application | Condition: status=under_review | Role: sn_scholar.provider
  - Server: set status=approved, trigger award creation, notify applicant
- [ ] UA: `Deny Application`
  - Table: sn_scholar_application | Condition: status=under_review | Role: sn_scholar.provider
  - Client: prompt for denial reason | Server: save reason, set status=denied, notify applicant
- [ ] UA: `Request More Documents`
  - Table: sn_scholar_application | Condition: status=under_review | Role: sn_scholar.provider
  - Client: prompt for doc list | Server: set status=draft, notify applicant with list
- [ ] UA: `Release Disbursement`
  - Table: sn_scholar_award | Condition: status=active AND disbursement_status=pending | Role: sn_scholar.provider
  - Server: set disbursement_status=released, notify scholar
- [ ] All 5 UI Actions tested with correct roles ✓
- [ ] Role-blocking verified (applicant cannot trigger provider actions) ✓

---

## PHASE 6 — Integration & Polish
> Owner: All members | Target: Day 6 (Saturday)

### End-to-End Test Cases
- [ ] TC01: Provider posts scholarship → visible in student catalog
- [ ] TC02: Student checks eligibility → AI result displayed correctly
- [ ] TC03: Student submits complete application → Flow triggered, provider notified
- [ ] TC04: Business Rule blocks incomplete submission (missing PSA)
- [ ] TC05: Provider approves → Award created, student notified
- [ ] TC06: Provider denies with reason → Student notified with reason
- [ ] TC07: Provider requests more docs → Student resubmits → Provider re-reviews
- [ ] TC08: Email inbound approval (reply APPROVE) → application approved
- [ ] TC09: Student submits compliance → Provider reviews → Compliant
- [ ] TC10: Non-compliant compliance → Award put On Hold → Scholar notified
- [ ] TC11: Role isolation — applicant cannot access provider pages
- [ ] TC12: Role isolation — provider cannot submit student applications

### Bug Fixes & Polish
- [ ] All TC failures resolved
- [ ] Service Portal mobile layout check (responsive)
- [ ] Email templates formatting verified (HTML renders correctly)
- [ ] Form field labels reviewed (clear, PH-context appropriate)
- [ ] Loading states on all async widget calls

### Compliance & Disbursement (P1)
- [ ] Compliance submission widget fully functional
- [ ] Compliance review widget on provider page
- [ ] Disbursement status field editable by provider on award record

### Source Control Cleanup
- [ ] All changes committed to GitHub
- [ ] No credentials or secrets in repo
- [ ] Clean commit history with meaningful messages
- [ ] Final push verified (Studio → Source Control → Push Changes)
- [ ] Update Set exported: `ScholarHub_v1.0_[date].xml`
- [ ] Update Set .xml saved to team shared folder

---

## PHASE 7 — Presentation Prep
> Owner: Member 5 | Target: Day 7 (Sunday)

### Presentation Deck (7 slides)
- [ ] Slide 1: Title — ScholarHub, team name, members, hackathon details
- [ ] Slide 2: Problem Statement — PH scholarship fragmentation pain points
- [ ] Slide 3: Key Challenges — 5 specific challenges with data references
- [ ] Slide 4: Proposed Solution — ScholarHub overview, 3 roles, 6 phases
- [ ] Slide 5: Process Flow Diagram — the full 6-phase flow (SVG exported as image)
- [ ] Slide 6: Key Benefits — student, provider, system-wide
- [ ] Slide 7: Demo — walkthrough slide or embedded video

### Demo Preparation
- [ ] Demo data seeded (3 scholarships, 2 applicant profiles ready)
- [ ] Demo script written (step-by-step 5-minute walkthrough)
- [ ] Backup demo video recorded (screen recording, 4-5 minutes)
- [ ] ServiceNow instance confirmed active (prevent PDI hibernation)
- [ ] Demo dry run with full team

### Final Deliverables Checklist
- [ ] ScholarHub ServiceNow instance accessible (URL shared with judges)
- [ ] GitHub repository accessible (URL shared)
- [ ] Update Set .xml ready to share
- [ ] Presentation deck .pptx finalized and exported
- [ ] Project Documentation .docx finalized
- [ ] Backup demo video .mp4 ready
- [ ] FEATURES.md final state committed to GitHub

---

## RUBRIC COVERAGE TRACKER
<!-- Agent: mark [x] when corresponding implementation is confirmed working -->
| Rubric Item | Weight | Implementation | Status |
|-------------|--------|----------------|--------|
| Business Rules | Dev 40% | 5 BRs: validate_docs, set_status, prevent_duplicate, flag_non_compliant, auto_close | [ ] |
| Client Scripts | Dev 40% | 4 CSs: income_fields, doc_checklist, gwa_indicator, lock_form | [ ] |
| UI Actions | Dev 40% | 5 UAs: Submit, Approve, Deny, RequestDocs, ReleaseDisbursement | [ ] |
| Notifications Outbound | Dev 40% | 9 templates covering all lifecycle events | [ ] |
| Notifications Inbound | Dev 40% | Inbound email action for email-based approval | [ ] |
| Approval via Email | Dev 40% | Inbound action parses APPROVE/DENY, triggers flow branch | [ ] |
| Integration Hub | Dev 40% | REST Action to Claude API with input/output variable mapping | [ ] |
| Flow Designer | Dev 40% | Full Application Lifecycle Flow with all branches | [ ] |
| Service Portal | 10% | Student view (6 pages) + Provider view (5 pages) with User Criteria | [ ] |
| User Criteria & Roles | Dev 40% | 3 roles + 11 widget-level criteria | [ ] |
| AI Integration | 10% | Claude API eligibility pre-screen via Integration Hub | [ ] |
| Bonus: Agentic AI | Bonus | AI result drives form behavior (show/hide fields) | [ ] |

---

## GATE STATUS
<!-- Agent: update after each session -->
| Gate | Last Run | Status |
|------|----------|--------|
| session-start (read this file + git status) | never | pending |
| pre-commit (secret scan + review staged) | never | pending |
| self-audit (audit, fix, re-audit) | never | pending |
| post-work (update this file + push) | never | pending |
| push-verify (confirm push + CI) | never | pending |
| session-handoff (write handoff block) | never | pending |

---

## BLOCKER LOG
<!-- Agent: log blockers immediately when encountered -->
| Date | Blocker | Owner | Resolved |
|------|---------|-------|----------|
| - | None yet | - | - |

---

## DEAD ENDS LOG
<!-- Agent: log ruled-out approaches so no session repeats them -->
| Date | Approach Tried | Why Abandoned | Alternative Used |
|------|----------------|---------------|-----------------|
| - | None yet | - | - |

---

## KNOWN GOTCHAS
<!-- Agent: add immediately when a non-obvious issue is discovered -->
- PDI instances hibernate after 10 days of inactivity — keep instance active before demo
- ServiceNow inbound email actions require reply-to address to match exactly
- Flow Designer wait conditions need polling interval configured (default is too slow for demo)
- GlideAjax in Client Scripts is async — always use callback pattern, never expect synchronous return
- Integration Hub credentials stored as Connection Alias — do NOT hardcode API keys in scripts
- User Criteria on widgets is evaluated at page load — test role switching by impersonating users

---

_Last updated: not started | Next session should: read this file first, check git status, then begin Phase 1_
