# AI Eligibility Logic — ScholarHub Pre-screen
> Agent: this defines exactly how the AI eligibility check works end-to-end for ScholarHub's Philippine scholarship context.

---

## What the AI Pre-screen Checks

Based on real Philippine scholarship requirements (DOST, CHED, SM Foundation, Landbank, Metrobank):

| Criteria | How to Check | Hard vs Soft |
|----------|-------------|--------------|
| GWA minimum | applicant.gwa >= scholarship.gwa_requirement | **HARD** — below = not_eligible |
| Family income cap | applicant.family_income <= scholarship.income_cap | **HARD** — over cap = not_eligible (for need-based) |
| Priority course match | applicant.course IN scholarship.course_priority | **SOFT** — mismatch = partial, not disqualifying |
| School type preference | applicant.school_type matches preference | **SOFT** — mismatch = partial |
| Honor rank | applicant.honor_rank is present if merit-based | **SOFT** — absent = partial |

**Hard criteria failure → `not_eligible`**
**Only soft criteria failures → `partial`**
**All criteria met → `eligible`**

---

## Prompt Construction Logic

```javascript
// AIEligibilityHelper Script Include method
buildPrompt: function() {
    var gwa = this.getParameter('sysparm_gwa');
    var income = this.getParameter('sysparm_income');
    var course = this.getParameter('sysparm_course');
    var schoolType = this.getParameter('sysparm_school_type');
    var honorRank = this.getParameter('sysparm_honor_rank');
    var scholarshipType = this.getParameter('sysparm_scholarship_type');
    var gwaReq = this.getParameter('sysparm_gwa_req');
    var incomeCap = this.getParameter('sysparm_income_cap');
    var coursePriority = this.getParameter('sysparm_course_priority');

    var systemPrompt = [
        'You are a scholarship eligibility assistant for Philippine scholarships.',
        'Assess eligibility based on the criteria and profile provided.',
        'Rules:',
        '- eligible: meets ALL criteria including GWA and income requirements',
        '- partial: meets SOME criteria but has gaps in soft requirements',
        '- not_eligible: fails on GWA minimum OR income cap (these are hard requirements)',
        'Return ONLY raw JSON, no markdown, no explanation.',
        'Format: {"status":"eligible|partial|not_eligible","matched":["..."],"missing":["..."],"note":"..."}'
    ].join('\n');

    var criteria = [];
    if (scholarshipType) criteria.push('Type: ' + scholarshipType);
    if (gwaReq) criteria.push('Minimum GWA: ' + gwaReq);
    if (incomeCap) criteria.push('Max annual family income (PHP): ' + incomeCap);
    if (coursePriority) criteria.push('Priority courses: ' + coursePriority);

    var profile = [];
    profile.push('GWA: ' + (gwa || 'not provided'));
    profile.push('Annual family income (PHP): ' + (income || 'not provided'));
    profile.push('Course: ' + (course || 'not provided'));
    profile.push('School type: ' + (schoolType || 'not provided'));
    if (honorRank) profile.push('Honor/rank: ' + honorRank);

    var userMessage = 'Scholarship criteria:\n' + criteria.join('\n') +
                      '\n\nApplicant profile:\n' + profile.join('\n') +
                      '\n\nAssess eligibility and return JSON only.';

    return JSON.stringify({
        system: systemPrompt,
        message: userMessage
    });
},
```

---

## Expected AI Outputs for Common Scenarios

### Scenario 1: Clearly Eligible (DOST-like)
```
Input: GWA=92, income=200000, course=BSCS, school=public, type=need_based, gwa_req=85, income_cap=360000
Output:
{
  "status": "eligible",
  "matched": [
    "GWA 92 exceeds minimum requirement of 85",
    "Annual income PHP 200,000 is within PHP 360,000 cap",
    "BSCS is a priority STEM course",
    "Public school background meets preference"
  ],
  "missing": [],
  "note": "You meet all criteria for this scholarship. Prepare your PSA, grades, and ITR."
}
```

### Scenario 2: Partial — Good GWA but income borderline
```
Input: GWA=88, income=480000, course=BSIT, school=private, type=need_based, gwa_req=85, income_cap=300000
Output:
{
  "status": "not_eligible",
  "matched": ["GWA 88 meets minimum requirement of 85"],
  "missing": ["Annual income PHP 480,000 exceeds the PHP 300,000 cap for need-based scholarships"],
  "note": "Your income exceeds the need-based requirement. Consider applying for merit-based scholarships."
}
```

### Scenario 3: Partial — Course not in priority list
```
Input: GWA=90, income=150000, course=BSN, school=public, type=combined, gwa_req=88, income_cap=500000, course_priority=BSIT,BSCS,BSECE
Output:
{
  "status": "partial",
  "matched": [
    "GWA 90 meets minimum requirement of 88",
    "Annual income PHP 150,000 is within PHP 500,000 cap"
  ],
  "missing": ["Nursing (BSN) is not in the priority courses: BSIT, BSCS, BSECE"],
  "note": "You meet the GWA and income requirements, but your course is not a priority. You may still apply — the provider makes the final decision."
}
```

---

## Portal Widget Flow (Step by Step)

```
1. User opens Scholarship Detail page
2. Widget loads: shows scholarship details + eligibility check form

3. User fills in:
   - GWA (numeric input)
   - Annual family income (numeric input, PHP)
   - Course/program (text input or dropdown)
   - School type (radio: public/private/science_hs)
   - Honor rank (optional text)

4. User clicks "Check My Eligibility"

5. Widget client controller:
   c.checkEligibility = function() {
       c.loading = true;
       c.aiResult = null;

       $http.post('/api/now/sp/widget/scholar-ai-check', {
           data: {
               scholarship_id: c.data.scholarshipId,
               gwa: c.eligForm.gwa,
               income: c.eligForm.income,
               course: c.eligForm.course,
               school_type: c.eligForm.schoolType,
               honor_rank: c.eligForm.honorRank
           }
       }).then(function(response) {
           c.loading = false;
           c.aiResult = response.data.result;
       }).catch(function() {
           c.loading = false;
           c.aiResult = { status: 'error', note: 'Check unavailable. You may still apply.' };
       });
   };

6. Widget displays result card (green/yellow/red)
   - Show matched[] as checkmarks
   - Show missing[] as warning items
   - Show note as advisory text
   - Keep "Apply Now" button always visible (AI result is advisory only)
```

---

## Storing and Displaying Cached Results

```javascript
// Widget server script — check for cached result
var applicationSysId = $sp.getParameter('application_id');
if (applicationSysId) {
    var appGR = new GlideRecord('sn_scholar_application');
    if (appGR.get(applicationSysId)) {
        var cached = appGR.getValue('ai_pre_screen_result');
        if (cached) {
            try {
                data.cachedAiResult = JSON.parse(cached);
            } catch(e) {
                data.cachedAiResult = null;
            }
        }
    }
}
```

---

## What the AI Does NOT Do

- Does NOT make the final approval decision (provider does)
- Does NOT verify document authenticity (manual review by provider)
- Does NOT access real Philippine government databases
- Does NOT guarantee approval even if result is "eligible"

Always display this disclaimer near the result:
> "This is an automated pre-screening only. The scholarship provider makes all final decisions."
