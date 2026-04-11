# Quality Contract — ScholarHub
> Agent: consult this during GATE 3 (self-audit). This defines what "done" looks like.

---

## What "Done" Means Per Feature Type

### Business Rule is DONE when:
- [ ] Script has no syntax errors (check sys_log after saving)
- [ ] Trigger type is correct (Before/After Insert vs Update)
- [ ] Tested with a real record using the correct role (impersonate test user)
- [ ] Error messages are user-friendly (no technical jargon)
- [ ] Edge cases tested: empty fields, wrong role, duplicate attempts
- [ ] Committed to GitHub with descriptive commit message

### Client Script is DONE when:
- [ ] Fires on the correct event (onChange/onLoad/onSubmit)
- [ ] Works for NEW records and EXISTING records (onLoad behaves differently)
- [ ] GlideAjax calls use proper callback pattern (no sync assumptions)
- [ ] Tested with Form Designer preview and real impersonated user
- [ ] Does not break other fields on the form
- [ ] No console errors in browser developer tools

### Flow Designer Step is DONE when:
- [ ] Step executes without errors (check Flow Engine log)
- [ ] Variables pass correctly between steps
- [ ] Branch conditions cover all possible states
- [ ] Wait condition has a timeout configured (don't leave it infinite)
- [ ] Tested end-to-end from trigger to completion

### Service Portal Widget is DONE when:
- [ ] Widget renders without JavaScript errors (check browser console)
- [ ] Server script returns expected data object
- [ ] Angular client controller updates UI reactively
- [ ] Works for the correct role (tested with impersonated user)
- [ ] Works on mobile viewport (check Chrome DevTools responsive view)
- [ ] Empty state handled (what shows when there are no records?)
- [ ] Error state handled (what shows if the API call fails?)

### Email Notification is DONE when:
- [ ] Template uses correct `${field_name}` tokens and they resolve
- [ ] Sent to the correct recipient (test with real test email address)
- [ ] Subject line is specific and actionable
- [ ] HTML renders correctly in email client (check both plain text and HTML)
- [ ] For inbound: reply processing tested with actual email reply

### Integration Hub Action is DONE when:
- [ ] REST call succeeds (check Integration Hub execution log)
- [ ] Request body constructs correctly with dynamic values
- [ ] Response parses correctly (no JSON parse errors)
- [ ] Error case handled (API down, rate limit, bad response)
- [ ] API key NOT hardcoded — uses Connection Alias credential
- [ ] Tested with real Claude API call (not just mocked)

---

## Quality Bar for Demo Day

By Day 6 end, ScholarHub must pass ALL of these:

### Functional Bar
- All 12 test cases in FEATURES.md Phase 6 pass without errors
- No errors visible in ServiceNow System Log during demo flow
- No JavaScript console errors visible during portal navigation
- All 9 email notifications send to test email addresses

### Performance Bar
- Scholarship catalog loads in under 3 seconds
- Application form loads in under 3 seconds
- AI pre-screen response received in under 10 seconds (Claude API latency)
- Flow Designer approval workflow completes in under 30 seconds

### Access Control Bar
- Applicant cannot see provider pages (test by impersonating applicant user)
- Provider cannot submit applications (test by impersonating provider user)
- Admin can access all records
- Unauthenticated user is redirected to login

### Portal Design Bar
- ScholarHub branding consistent (teal #0F6E56, blue #0C447C)
- Mobile viewport does not break layout
- All buttons have clear labels (no "Button 1" or "Submit Form")
- Empty states show friendly messages ("No applications yet" not blank screen)

---

## The Self-Audit Protocol (GATE 3)

Run these steps in order after completing any feature:

```
1. Read the code fresh — does it actually do what was intended?
2. What happens if: a field is empty? The wrong user? A network error?
3. Open ServiceNow System Log — any errors since you saved?
4. Impersonate the correct test user — test the feature
5. Fix any issues found
6. Re-run steps 1–4 (the re-audit catches bugs introduced by the fix)
7. Only mark [x] in FEATURES.md after passing the re-audit
```

---

## Known Risk Areas (Require Extra Audit Attention)

- **Flow Designer wait conditions** — infinite waits will stall demo; always set a timeout
- **Inbound email parsing** — test with actual email reply, not just internal testing
- **Business Rule abort actions** — test that `setAbortAction(true)` actually stops the record insert
- **User Criteria on widgets** — test with impersonation, not just admin view
- **Integration Hub credentials** — API key must be in Connection Alias, not in script
- **PDI hibernation** — instance sleeping during demo is a showstopper; log in morning of demo
