# Handoff Template — ScholarHub Session Handoff Schema
> Locked template. Every session ends with this. No shortcuts, no empty fields.
> "None" is a valid value. A blank field is not.

---

## HOW TO USE

At the end of every session, the agent produces this block. The next session pastes it verbatim to start. This is the bridge across the no-shared-memory gap.

**Do not wait to be asked.** When a natural session breakpoint is reached (a feature shipped, a blocker hit, context getting heavy), proactively say: "I recommend we start a fresh session. Here's the handoff."

---

## HANDOFF SCHEMA (Copy and fill)

```
═══════════════════════════════════════════════════════
SCHOLARHUB SESSION HANDOFF
Generated: [DATE TIME]
Member: [MEMBER NAME / MEMBER NUMBER]
Day of Sprint: [Day 1–7]
═══════════════════════════════════════════════════════

STATUS:
[One sentence — what state is ScholarHub in right now?
Example: "Phase 2 BRs complete; Flow Designer started but not tested"]

───────────────────────────────────────────────────────
LAST SESSION:
───────────────────────────────────────────────────────
[Bullet list of what was specifically accomplished. Be precise.]
- Example: "BR sn_scholar_validate_documents implemented — blocks missing docs ✓"
- Example: "BR sn_scholar_set_status_on_submit implemented — stamps date, triggers flow ✓"
- Example: "Committed: [abc1234] feat: add validate and set-status business rules"
- Example: "FEATURES.md updated: Phase 2 tasks 1–4 marked [x]"

───────────────────────────────────────────────────────
IN FLIGHT:
───────────────────────────────────────────────────────
[What is half-done? Be specific about where it stopped.]
- Example: "Flow Designer Step 3 (wait condition) — created trigger, action steps 1-2 done, wait step not configured yet"
- Example: "Email Template T3 — HTML written, not yet saved as notification record"
- If nothing: "Clean slate — all started tasks are complete"

───────────────────────────────────────────────────────
BLOCKERS:
───────────────────────────────────────────────────────
[What needs a human decision or external action?]
- Example: "Need Anthropic API key from team lead before Integration Hub can be tested"
- Example: "PDI instance requires re-activation — login needed before next session"
- If none: "None"

───────────────────────────────────────────────────────
NEXT CANDIDATES: (ranked by priority)
───────────────────────────────────────────────────────
1. [Highest priority — specific and scoped]
   Example: "Implement Flow Designer wait condition for approval — ~2 hours"
2. [Second option]
   Example: "Create email notification templates T1 and T2 — ~1 hour"
3. [Third option]
   Example: "Begin Service Portal catalog widget — ~3 hours"

───────────────────────────────────────────────────────
FEATURES.md UPDATES:
───────────────────────────────────────────────────────
[Which tasks were marked done / in-progress / blocked this session?]
- Example: "Phase 2 — Business Rules: tasks 1, 2, 3 marked [x]"
- Example: "Phase 2 — Script Includes: ScholarUtils marked [~] (started, not complete)"
- Example: "OVERALL PROGRESS updated: 8/103 done (7%)"

───────────────────────────────────────────────────────
GATE CHECKS COMPLETED:
───────────────────────────────────────────────────────
[x] session-start
[ ] pre-commit      [note: no commits this session / or: committed [hash]]
[x] self-audit      [note: re-audited after fixing X]
[x] post-work       [note: FEATURES.md updated]
[x] push-verify     [note: pushed, GitHub shows [hash] as latest]
[x] session-handoff [note: this block]

───────────────────────────────────────────────────────
SESSION NOTES:
───────────────────────────────────────────────────────
[Anything the next session should know that doesn't fit above]
- Dead ends explored: [e.g., "Tried using GlideRecord in Client Script directly — blocked by ServiceNow security, use GlideAjax instead. Added to KNOWN GOTCHAS."]
- Surprises: [e.g., "PDI instance requires re-login every 24h — not just 10 days"]
- Decisions deferred: [e.g., "Inbound email reply-to address not decided — need team input"]
- Performance notes: [e.g., "AI pre-screen takes ~3s — added loading spinner to widget"]

═══════════════════════════════════════════════════════
NEXT SESSION KICKOFF CHECKLIST:
1. Read this block fully
2. Run GATE 1 (session-start): read FEATURES.md, run git status
3. Verify IN FLIGHT items still match reality
4. Pick from NEXT CANDIDATES or address BLOCKERS first
5. State your plan before writing code
═══════════════════════════════════════════════════════
```

---

## EXAMPLE HANDOFF (Day 2 End, Member 1)

```
═══════════════════════════════════════════════════════
SCHOLARHUB SESSION HANDOFF
Generated: 2026-05-13 18:45
Member: Member 1 (Backend Lead)
Day of Sprint: Day 2
═══════════════════════════════════════════════════════

STATUS:
All 5 Business Rules implemented and tested. Script Includes started
(ScholarUtils complete, AIEligibilityHelper 50% done). Phase 2 is ~80% done.

LAST SESSION:
- BR sn_scholar_validate_documents: implemented and tested (missing PSA blocked ✓)
- BR sn_scholar_set_status_on_submit: implemented and tested (date stamped ✓)
- BR sn_scholar_prevent_duplicate: implemented and tested (second application blocked ✓)
- BR sn_scholar_flag_non_compliant: implemented, test pending (needs compliance table data)
- BR sn_scholar_auto_close_scholarships: scheduled job configured, not yet tested
- Script Include ScholarUtils: getRequiredDocs() and checkDuplicate() complete
- Committed: [d7e4a12] feat: add all 5 business rules and ScholarUtils

IN FLIGHT:
- AIEligibilityHelper Script Include: buildPrompt() done, parseResponse() NOT done
  File: x_snc_scholar.AIEligibilityHelper, method parseResponse() still empty

BLOCKERS:
- BR sn_scholar_flag_non_compliant needs test data in sn_scholar_compliance
  (Member 2's Flow Designer work creates compliance records — wait for Day 3)

NEXT CANDIDATES:
1. Complete AIEligibilityHelper.parseResponse() — ~30 min
2. Test sn_scholar_flag_non_compliant once Member 2 has compliance data — ~30 min
3. Support Member 3 with any portal data queries they need — as needed

FEATURES.md UPDATES:
- Phase 2 Business Rules: tasks 1–5 all marked [x] (testing pending for #4 noted)
- Phase 2 Script Includes: ScholarUtils marked [x], AIEligibilityHelper marked [~]
- OVERALL PROGRESS updated: 22/103 done (21%)

GATE CHECKS COMPLETED:
[x] session-start
[x] pre-commit (committed d7e4a12)
[x] self-audit (re-audited BR validate_docs after fixing abort message typo)
[x] post-work (FEATURES.md updated and committed)
[x] push-verify (GitHub shows d7e4a12 as latest on main)
[x] session-handoff (this block)

SESSION NOTES:
- GOTCHA DISCOVERED: GlideRecord.addQuery() must be called BEFORE .query() — 
  obvious in hindsight but wasted 20 min. Added to KNOWN GOTCHAS in FEATURES.md.
- The sn_scholar_auto_close BR fires as a scheduled job — need to manually trigger 
  it in testing (System Scheduler → right-click → Execute Now). Not intuitive.
- Member 2 should note: Flow trigger on sn_scholar_application uses "status = submitted"
  as condition — make sure the BR sets this BEFORE the flow trigger fires (After Insert works).

═══════════════════════════════════════════════════════
NEXT SESSION KICKOFF CHECKLIST:
1. Read this block fully
2. Run GATE 1: read FEATURES.md, run git status
3. Verify AIEligibilityHelper is still in-flight at parseResponse()
4. Address blocker (flag_non_compliant test) once Member 2 has data
5. State your plan before writing code
═══════════════════════════════════════════════════════
```
