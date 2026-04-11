# Project Brief — ScholarHub
> Read this first, every session. This is the single source of truth for what we're building.

---

## What Is ScholarHub

ScholarHub is a **ServiceNow Scoped Application** that digitizes and centralizes the scholarship lifecycle for Philippine scholarship providers and student applicants. It eliminates the fragmented, paper-based, email-thread-driven process that causes qualified students to miss opportunities and providers to lose efficiency.

Built for: **ALPS 2.0 Final Evaluation Hackathon**
Partners: **CIT-U × EY GDS × ServiceNow**
Presentation: **May 21–23, 2026**
Sprint: **7 days, 5 members, AI-assisted**

---

## The Problem We're Solving

In the Philippines, scholarships (DOST, CHED, SM Foundation, Landbank, Metrobank, Aboitiz, university-internal) are managed through:
- Scattered social media posts and bulletin boards for discovery
- Paper forms and email threads for applications
- Spreadsheets for tracking applicants
- No standard workflow for review or approval
- No compliance monitoring for continuing scholars

**Result:** Qualified students miss scholarships. Providers can't scale. No audit trail.

---

## The Three Roles

| Role | ServiceNow Role ID | Who They Are |
|------|--------------------|--------------|
| Student / Applicant | `sn_scholar.applicant` | SHS graduates and college students seeking financial support |
| Scholarship Provider | `sn_scholar.provider` | Universities, CHED, DOST, SM Foundation, Landbank, Metrobank, foundations |
| Admin | `sn_scholar.admin` | IT admin or scholarship office head managing the platform |

**No other roles exist in this system.** Do not create additional roles without explicit instruction.

---

## The Six Phases (End-to-End Flow)

```
Phase 1: SETUP         Provider posts scholarship listing
Phase 2: DISCOVERY     Student browses, AI pre-screens eligibility
Phase 3: APPLICATION   Student submits application + uploads PH documents
Phase 4: REVIEW        Provider reviews, approves or denies
Phase 5: AWARD         System records award, notifies scholar
Phase 6: COMPLIANCE    Scholar submits per-semester docs, provider reviews, releases disbursement
```

---

## The Five Tables

```
sn_scholar_scholarship   Scholarship listings posted by providers
sn_scholar_application   Student applications (one per student per scholarship)
sn_scholar_document      Documents uploaded per application
sn_scholar_award         Approved scholarship grants
sn_scholar_compliance    Per-semester compliance submissions
```

Full field definitions: see **data-model.md**

---

## The Stack

| Layer | Technology |
|-------|-----------|
| Platform | ServiceNow (Scoped Application, scope: `x_snc_scholar`) |
| Workflow | ServiceNow Flow Designer |
| Portal | ServiceNow Service Portal (Angular.js widgets) |
| Server scripts | GlideRecord, Script Includes, Business Rules |
| Client scripts | g_form API, GlideAjax |
| External AI | Anthropic Claude API (claude-haiku-4-5-20251001) |
| Integration | ServiceNow Integration Hub (REST steps) |
| Source control | GitHub (linked via ServiceNow Studio) |
| Instance | ServiceNow Personal Developer Instance (PDI) |

---

## Hackathon Rubric Weights

| Component | Weight | ScholarHub Implementation |
|-----------|--------|--------------------------|
| Development (BRs, Client Scripts, Integration, Components) | 40% | 5 BRs, 4 CSs, 5 UAs, Flow, Notifications, Integration Hub |
| Solution to the Problem | 20% | PH scholarship lifecycle, AI pre-screen |
| AI Solution | 10% | Claude API eligibility pre-screen |
| Service Portal Design | 10% | Student view + Provider view |
| Presentation / Documentation | 10% | 7-slide deck + this doc set |
| Participation / Involvement | 10% | All 5 members active |
| Bonus | +points | Agentic AI or Welcome to ServiceNow microcert |

---

## Team Structure

| Member | Role | Primary Days |
|--------|------|-------------|
| Member 1 | Backend Lead | Days 1–2: Tables, BRs, Script Includes |
| Member 2 | Workflow Lead | Day 3: Flow Designer, Notifications |
| Member 3 | Portal Lead | Days 3–4: Service Portal pages + widgets |
| Member 4 | AI + Integration | Day 5: Integration Hub, AI widget, Client Scripts |
| Member 5 | UI + Docs + Deck | Day 5: UI Actions, CS polish, Day 7: Slides |

---

## What "Done" Means for This Project

A feature is **done** when:
1. The code is implemented in ServiceNow
2. It has been tested with the correct test user (impersonating the right role)
3. It has been committed and pushed to GitHub
4. FEATURES.md has been updated with [x] status

A demo is **ready** when:
1. The 5-step demo walkthrough completes without errors
2. All 11 rubric items show [x] in FEATURES.md Rubric Coverage Tracker
3. ServiceNow instance is accessible and seeded with demo data
4. Backup demo video exists as .mp4

---

## Critical Constraints

- **1 week only** — no scope creep, no unrequested features
- **Philippine context** — document types are PSA, ITR, grades, good moral, photos. Not US-centric.
- **No real financial processing** — disbursement is a status field only
- **No SMS** — email only for notifications
- **No live PSA/CHED API** — document verification is manual review by provider
- **PDI instance hibernates** — log in daily to keep it active
