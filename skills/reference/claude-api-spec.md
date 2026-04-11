# Claude API Specification — ScholarHub AI Integration
> Agent: every detail needed to call the Anthropic API correctly. Do not deviate from these specs.

---

## API Endpoint

```
URL:     https://api.anthropic.com/v1/messages
Method:  POST
```

---

## Required Headers

```
Content-Type:      application/json
x-api-key:         [your API key — sk-ant-...]
anthropic-version: 2023-06-01
```

---

## Request Body Structure

```json
{
  "model": "claude-haiku-4-5-20251001",
  "max_tokens": 500,
  "system": "[system prompt string]",
  "messages": [
    {
      "role": "user",
      "content": "[user message string]"
    }
  ]
}
```

**Model to use:** `claude-haiku-4-5-20251001`
- Fast response time (important for real-time eligibility check)
- Cost-efficient (demo budget friendly)
- Sufficient capability for structured eligibility assessment

**Max tokens:** 500 — enough for a structured JSON eligibility response.

---

## System Prompt (ScholarHub Eligibility Pre-screen)

```
You are a scholarship eligibility assistant for Philippine scholarships.

Given scholarship criteria and an applicant profile, assess eligibility.

Rules:
- eligible: applicant meets ALL criteria
- partial: applicant meets SOME criteria but has gaps
- not_eligible: applicant fails on one or more HARD requirements (GWA or income)

Return ONLY a valid JSON object. No explanation, no markdown, no code fences. Just raw JSON.

Required format:
{
  "status": "eligible" | "partial" | "not_eligible",
  "matched": ["list of criteria the applicant meets"],
  "missing": ["list of requirements the applicant does not meet"],
  "note": "one-sentence advisory note for the applicant"
}
```

---

## User Message Template (Dynamic)

Build this dynamically from the scholarship and applicant data:

```javascript
function buildEligibilityPrompt(gwa, income, course, schoolType, honorRank,
                                 scholarshipType, gwaReq, incomeCap, coursePriority) {
    var criteria = [];
    if (gwaReq) criteria.push('Minimum GWA: ' + gwaReq);
    if (incomeCap) criteria.push('Maximum annual family income: PHP ' + incomeCap);
    if (scholarshipType) criteria.push('Scholarship type: ' + scholarshipType);
    if (coursePriority) criteria.push('Priority courses: ' + coursePriority);
    if (schoolType && schoolType != 'both') criteria.push('School type preference: ' + schoolType);

    var profile = [];
    profile.push('GWA: ' + (gwa || 'not provided'));
    profile.push('Annual family income: PHP ' + (income || 'not provided'));
    profile.push('Course/Program: ' + (course || 'not provided'));
    profile.push('School type: ' + (schoolType || 'not provided'));
    if (honorRank) profile.push('Honor rank: ' + honorRank);

    return 'Scholarship criteria:\n' + criteria.join('\n') +
           '\n\nApplicant profile:\n' + profile.join('\n') +
           '\n\nAssess eligibility and return JSON only.';
}
```

---

## Response Structure

```json
{
  "id": "msg_...",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "{\"status\":\"eligible\",\"matched\":[\"GWA 90 meets minimum 85\",\"Income PHP 200,000 is within PHP 300,000 cap\"],\"missing\":[],\"note\":\"You appear to meet all criteria for this scholarship.\"}"
    }
  ],
  "model": "claude-haiku-4-5-20251001",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 234,
    "output_tokens": 87
  }
}
```

**Parsing the response:**
```javascript
var responseBody = /* raw JSON string from API */;
var parsed = JSON.parse(responseBody);
var rawText = parsed.content[0].text;  // this is the JSON string Claude returned
var result = JSON.parse(rawText);      // parse Claude's JSON output

var status = result.status;            // 'eligible', 'partial', 'not_eligible'
var matched = result.matched;          // array of strings
var missing = result.missing;          // array of strings
var note = result.note;                // string
```

---

## Error Codes

| HTTP Status | Meaning | Action |
|-------------|---------|--------|
| 200 | Success | Parse normally |
| 400 | Bad request (malformed JSON) | Check request body format |
| 401 | Unauthorized | API key is wrong or missing |
| 403 | Forbidden | API key doesn't have access |
| 422 | Unprocessable | Model name wrong or params invalid |
| 429 | Rate limited | Wait 60 seconds and retry |
| 500 | Server error | Retry after 1-2 minutes |
| 529 | Overloaded | Retry with exponential backoff |

---

## Error Handling Pattern

```javascript
try {
    var response = rm.execute();
    var statusCode = response.getStatusCode();
    var body = response.getBody();

    if (statusCode == 200) {
        var apiResponse = JSON.parse(body);
        var resultText = apiResponse.content[0].text;
        // Remove any accidental markdown fences
        resultText = resultText.replace(/```json|```/g, '').trim();
        var result = JSON.parse(resultText);

        data.status = result.status || 'not_eligible';
        data.matched = result.matched || [];
        data.missing = result.missing || [];
        data.note = result.note || '';

    } else if (statusCode == 429) {
        data.status = 'error';
        data.note = 'Eligibility check is temporarily busy. Please try again in a moment.';

    } else {
        data.status = 'error';
        data.note = 'Eligibility check unavailable (HTTP ' + statusCode + '). You may still apply manually.';
        gs.log('ScholarHub AI: HTTP ' + statusCode + ' - ' + body, 'ScholarHub');
    }

} catch(e) {
    data.status = 'error';
    data.note = 'Could not complete eligibility check. You may still apply manually.';
    gs.log('ScholarHub AI Error: ' + e.message, 'ScholarHub');
}
```

---

## Portal Display Logic

```javascript
// In widget client controller:
c.getStatusConfig = function(status) {
    var configs = {
        'eligible': {
            label: 'You appear eligible for this scholarship',
            cssClass: 'scholar-eligible',
            icon: 'check-circle'
        },
        'partial': {
            label: 'You may qualify — review the gaps below',
            cssClass: 'scholar-partial',
            icon: 'exclamation-circle'
        },
        'not_eligible': {
            label: 'You do not meet the criteria for this scholarship',
            cssClass: 'scholar-not-eligible',
            icon: 'times-circle'
        },
        'error': {
            label: 'Eligibility check unavailable — you may still apply',
            cssClass: 'scholar-error',
            icon: 'info-circle'
        }
    };
    return configs[status] || configs['error'];
};
```

```css
/* Widget CSS */
.scholar-eligible    { background: #E1F5EE; border-left: 4px solid #0F6E56; color: #085041; }
.scholar-partial     { background: #FAEEDA; border-left: 4px solid #BA7517; color: #633806; }
.scholar-not-eligible{ background: #FCEBEB; border-left: 4px solid #A32D2D; color: #501313; }
.scholar-error       { background: #F1EFE8; border-left: 4px solid #5F5E5A; color: #2C2C2A; }
```

---

## Caching the AI Result

Store the last result on the application record to avoid repeated API calls:

```javascript
// After getting result, store on application record
var appGR = new GlideRecord('sn_scholar_application');
if (appGR.get(applicationSysId)) {
    appGR.setValue('ai_pre_screen_result', JSON.stringify({
        status: result.status,
        matched: result.matched,
        missing: result.missing,
        note: result.note,
        checked_at: new GlideDateTime().getDisplayValue()
    }));
    appGR.update();
}

// On reload, read from stored result instead of calling API again
var stored = gr.getValue('ai_pre_screen_result');
if (stored) {
    data.aiResult = JSON.parse(stored);
    data.aiFromCache = true;
}
```
