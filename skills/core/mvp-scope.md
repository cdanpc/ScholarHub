# MVP & Scope — ScholarHub
> Agent: if a feature is not on this list, do not build it. No unrequested complexity.

---

## P0 — Must Have (Demo Blockers)

Every P0 item must be working before Day 6 demo prep starts.

| # | Feature | Why Non-Negotiable |
|---|---------|-------------------|
| 1 | Scholarship catalog (post + browse) | Without it, nothing else works |
| 2 | AI eligibility pre-screen | Directly scores AI Integration (10%) + bonus potential |
| 3 | Application form + dynamic document checklist | Core applicant action, shows Client Scripts |
| 4 | BR: Validate documents on submit | Development rubric (40%) — BRs are scored |
| 5 | BR: Auto-set status + stamp date on submit | Development rubric — BRs are scored |
| 6 | Flow Designer approval workflow | Development rubric — Flow Designer is scored |
| 7 | Outbound email notifications (all 9) | Development rubric — Notifications required |
| 8 | Inbound email approval | Development rubric — Inbound Notifications required |
| 9 | UI Actions: Submit, Approve, Deny, Request Docs | Development rubric — UI Actions required |
| 10 | Service Portal (student view + provider view) | Service Portal Design (10%) directly scored |
| 11 | 3-role access control + User Criteria | Development rubric — Access Roles required |
| 12 | Integration Hub → Claude API | Development rubric — Integration Hub required |

---

## P1 — Should Have (Add on Day 5–6 if time allows)

| Feature | Why Important |
|---------|--------------|
| Compliance submission form (basic) | Completes Phase 6 story for judges |
| Disbursement status field on award | Closes the award management loop |
| Release Disbursement UI Action | Lets provider complete the award lifecycle |
| Compliance review widget (provider) | Gives provider the full picture |

---

## P2 — Nice to Have (Day 6 only if everything else is done)

| Feature | Note |
|---------|------|
| Richer AI eligibility hint cards (expandable) | Improves UX but not scored separately |
| Scholarship search/filter (by type, GWA, course) | Nice portal feature |
| Application history timeline widget | Visual polish |

---

## Out of Scope — Do Not Build

If asked to build any of these, decline and redirect to MVP priorities:

| Feature | Reason |
|---------|--------|
| Actual financial disbursement processing | Requires banking integration, out of scope |
| Live PSA / CHED database verification | APIs not accessible in PDI demo environment |
| Mobile application | Time constraint; portal is mobile-responsive |
| Multi-language support (Filipino/Tagalog) | Post-hackathon enhancement |
| Automated scholarship renewal | Complex lifecycle logic, deferred to post-MVP |
| Analytics dashboard / reporting | Not scored; use default ServiceNow reports |
| Public scholarship listing (no login) | Outside portal scope for this sprint |
| SMS notifications | Requires Semaphore PH or similar; email is sufficient |
| Document OCR / auto-fill | Too complex for 1 week |
| Scholarship recommendation engine | Out of scope |
| Chat / messaging between applicant and provider | Out of scope |

---

## The Minimum Demo Walkthrough (5 Steps)

This is what judges MUST see. Every step must work without errors.

```
Step 1: Provider logs in → posts a new scholarship listing
        (type: need-based, GWA: 85, income cap: 300,000, course: BSIT)

Step 2: Student logs in → browses catalog → clicks scholarship
        → fills eligibility form → AI pre-screen result shown (green/yellow/red)

Step 3: Student fills application form → uploads PSA + grades + ITR
        → clicks Submit → application goes to provider queue

Step 4: Provider sees notification email → opens application queue in portal
        → reviews application → clicks Approve
        (or: replies APPROVE to email)

Step 5: System auto-creates Award record
        Student sees notification email → logs in → sees approved status + award details
```

**Demo data required before presentation:**
- 3 open scholarships (variety of types and criteria)
- 1 applicant profile pre-filled (GWA 90, income 200,000, BSIT, public school)
- 1 provider account ready to receive notifications
- All 9 email notification templates using test email addresses

---

## Scope Creep Warning Signs

If you find yourself doing any of the following, stop and re-read this file:

- Adding a 4th user role
- Building a real-time chat feature
- Creating report dashboards with charts
- Adding more than 1 external API integration
- Building a mobile-specific layout
- Creating automated scholarship matching/recommendations
- Integrating with any Philippine government database
- Adding file preview/viewer in the portal (link to file is sufficient)
