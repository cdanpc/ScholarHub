# Session Gates — ScholarHub Agent Discipline
> Source: Claude Code Field Manual (forged across 30+ real sessions)
> These are non-negotiable checkpoints. A session that skips gates ships bad code.

---

## GATE 1 — session-start
> Run this before writing a single line of code.

**Every new session begins here. No exceptions.**

```
1. Read FEATURES.md — check overall progress, last session summary, any blockers
2. Read the relevant skill file for today's work (e.g., flow-designer-guide.md for Day 3)
3. Run git status — verify working tree state matches what FEATURES.md says
4. Run git log --oneline -10 — confirm last commits are what you expect
5. Check ServiceNow instance is accessible (hit the instance URL)
6. Verify you know who you are (which team member / which day of sprint)
7. Trust live state over memory — every claim from the last session is a hypothesis until verified
```

**Do not skip this because "you know what the project state is."** Memory is a point-in-time snapshot. The repo is truth.

---

## GATE 2 — pre-commit
> Run before every git commit from ServiceNow Studio.

```
1. Secret scan the diff — no API keys, no passwords, no .env content in committed files
2. Confirm changes are in the correct scope (sn_scholar_ prefix, not global scope)
3. Review staged files for unexpected additions (system logs, temp files, credentials)
4. Write a meaningful commit message: "feat: add validate_documents business rule" not "update"
5. Never bypass Studio's sync hooks unless explicitly authorized by team lead
```

**The cost of one bad commit (leaked API key) lasts forever in git history.**

---

## GATE 3 — self-audit
> Run after completing any feature. One pass is never enough.

```
1. Does the code actually do what was intended? Read it fresh.
2. Does it handle the error case? (API timeout, missing field, wrong role)
3. Are there console errors or System Log errors in ServiceNow?
4. Did the Business Rule fire correctly? (check sys_log for script errors)
5. Does the Flow work end-to-end with test users? (not just individual steps)
6. Re-audit after fixing issues — the fix often introduces a new bug
7. Do NOT mark a task [x] done in FEATURES.md until it survives the second audit
```

**Done = survives the re-audit. Not "it ran once without errors."**

---

## GATE 4 — post-work
> Run before ending any work block (even a short one).

```
1. Update FEATURES.md — mark tasks [x] done, [~] in-progress, [!] blocked
2. Update FEATURES.md OVERALL PROGRESS counts
3. Update GATE STATUS table with today's date
4. If a new gotcha was discovered: add to KNOWN GOTCHAS section
5. If an approach was ruled out: add to DEAD ENDS LOG
6. Commit and push all changes to GitHub
7. Verify push landed (check GitHub repo in browser or via API)
```

**Memory drift (FEATURES.md out of sync with reality) is worse than no tracker.**

---

## GATE 5 — push-verify
> Run after every push to GitHub.

```
1. Confirm push succeeded — check Studio Source Control status shows "up to date"
2. Verify working tree is clean (no uncommitted changes remaining)
3. Open GitHub repo and confirm latest commit is visible with correct message
4. If this is a release/milestone: export Update Set and save as ScholarHub_v[x]_[date].xml
5. Check no sensitive files were included (re-scan last commit diff)
```

**Local and remote must agree. Asymmetry between Studio and GitHub is where releases break.**

---

## GATE 6 — session-handoff
> Run at the end of every session. Write the handoff block before closing anything.

Copy this template and fill it in. The next session pastes this to start:

```
STATUS: [one line — what state is ScholarHub in right now]

LAST SESSION:
- [specific task completed, e.g., "BR sn_scholar_validate_documents implemented and tested"]
- [commits made: [commit hash] — [message]]
- [any FEATURES.md tasks marked done]

IN FLIGHT:
- [what is half-done — specific file, table, widget, or flow step]
- [if nothing: "clean slate"]

BLOCKERS:
- [anything waiting on human decision or external dependency]
- [if none: "none"]

NEXT CANDIDATES: (ranked)
1. [highest priority next task — be specific, e.g., "implement Flow Designer Step 2 wait condition"]
2. [second option]
3. [third option]

MEMORY UPDATES:
- [which skill files were read or should be updated]
- [any new gotchas added to FEATURES.md]

GATE CHECKS:
- [x] session-start
- [x] pre-commit (if commits made)
- [x] self-audit
- [x] post-work
- [x] push-verify (if pushed)
- [ ] session-handoff (completing now)

SESSION NOTES:
- [dead ends explored, surprises, decisions deferred, anything next session should know]
```

**This is the bridge across the no-shared-memory gap. Without it, the next session starts from zero.**

---

## STANDING RULES (Always Active)

### Rule α — One task at a time
Finish the current task before proposing the next. For any action that requires a team member to do something manually (paste a credential, click a UI, connect hardware), give a numbered step list and wait for confirmation before continuing.

### Rule β — No unrequested complexity
Do not add helper utilities, abstractions, or backward-compatibility shims that were not asked for. Match scope of changes to what was asked. Three similar script includes are better than one premature abstraction.

### Rule γ — Confirm risky actions first
Before: deleting records, force-pushing to GitHub, modifying Production instances, changing global scope settings, or anything visible to all users — pause and ask. Authorization for one action does not extend to related actions.

---

## RECURRING DISCIPLINE (Scheduled)

### Every 5 sessions:
- Scan FEATURES.md for stale entries (tasks marked in-progress for >2 days)
- Scan KNOWN GOTCHAS for items that no longer apply after refactors
- Check GitHub repo for any uncommitted Studio changes

### Every day of sprint:
- Check ServiceNow PDI is still active (prevent hibernation — log in and save any record)
- Sync Studio with GitHub before starting new work (Source Control → Apply Remote Changes)

### Before Day 6 demo prep:
- Run all 12 test cases from FEATURES.md Phase 6
- Export Update Set as fresh .xml
- Confirm all 11 rubric items have [x] status in FEATURES.md Rubric Coverage Tracker

### Before final push (Day 7):
- Scan entire repo for: API keys, passwords, .env content, personal emails
- Verify commit history is clean and meaningful
- Tag the release: `git tag v1.0-hackathon` in Studio
