# Recurring Discipline — ScholarHub
> Agent: do these on schedule without being asked.

---

## Every Session (Daily During Sprint)

```
1. GATE 1 (session-start): Read FEATURES.md, run git status
2. Check PDI instance is accessible — log in and perform one action to prevent hibernation
3. Pull latest changes from GitHub before starting work:
   Studio → Source Control → Apply Remote Changes
4. GATE 6 (session-handoff): Produce handoff block before ending
```

---

## Every 5 Sessions

```
1. Scan FEATURES.md for stale [~] entries (in-progress > 2 days = likely blocked)
2. Scan KNOWN GOTCHAS — remove any that no longer apply after recent changes
3. Scan DEAD ENDS LOG — confirm these are still dead ends
4. Check GitHub repo: verify no uncommitted Studio changes lingering
5. Report: "FEATURES.md shows X% complete. [N] items stale. Recommend [action]."
```

---

## Day 6 (Pre-Demo Checklist)

```
1. Run all 12 test cases from FEATURES.md Phase 6
2. Fix every failing test before moving on
3. Check all 11 rubric items show [x] in FEATURES.md Rubric Coverage Tracker
4. Export Update Set: ScholarHub_v1.0_[date].xml
5. Seed demo data (3 scholarships, 2 applicant profiles, 1 provider account)
6. Confirm all 9 email templates use real test email addresses (not student names)
7. Record backup demo video (.mp4, 4-5 minutes)
8. Commit FEATURES.md with current state
```

---

## Day 7 (Pre-Presentation Final Check)

```
1. Verify ServiceNow instance is awake and accessible
2. Verify GitHub repository is accessible with clean history
3. Confirm Update Set .xml is ready and shareable
4. Run the 5-step demo walkthrough one final time
5. Final commit and push with tag: v1.0-hackathon
6. Confirm FEATURES.md is committed (judges may see it in the repo)
```
