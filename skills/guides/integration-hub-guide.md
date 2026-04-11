# Integration Hub Guide — ServiceNow
> Agent: Integration Hub connects ServiceNow to external APIs. ScholarHub uses it to call Claude API.

---

## Core Concepts

| Term | Definition |
|------|-----------|
| Connection Alias | A named connection to an external service (stores base URL + credential) |
| Credential | Auth details (API key, OAuth token, Basic Auth) stored securely |
| REST Message | A configured REST API call (URL, headers, method, body) |
| HTTP Method | A specific verb on a REST Message (GET, POST, PUT, DELETE) |
| Flow Action | A reusable Integration Hub action callable from Flow Designer |
| Spoke | A pre-built collection of actions for a service (e.g., GitHub Spoke, Slack Spoke) |

---

## Setting Up the Claude API Integration (Step by Step)

### Step 1: Create the Credential

```
Navigate to: Integration Hub → Connections & Credentials → Credentials → New
  OR: /sys_alias_list.do → New

Type: API Key Auth  (or Basic Auth with custom header approach)
Name: Anthropic Claude API Key
API Key Header Name: x-api-key
API Key: [your Claude API key — sk-ant-...]

Save.
```

### Step 2: Create the Connection

```
Navigate to: Integration Hub → Connections & Credentials → Connections → New

Name: Anthropic Claude
Connection URL: https://api.anthropic.com
Credential: [select: Anthropic Claude API Key]

Save.
```

### Step 3: Create the REST Message

```
Navigate to: System Web Services → Outbound → REST Messages → New

Name: claude_messages
Endpoint: https://api.anthropic.com/v1/messages
Authentication type: No authentication (we'll add headers manually)

After saving, add HTTP Headers:
  Header 1: Content-Type = application/json
  Header 2: anthropic-version = 2023-06-01
  Header 3: x-api-key = [use variable: ${api_key}]

Save.
```

### Step 4: Create the HTTP Method

```
On the REST Message record → HTTP Methods tab → New

Name: check_eligibility
HTTP Method: POST
Endpoint: [leave blank — inherits from REST Message]

Request body (template):
{
  "model": "claude-haiku-4-5-20251001",
  "max_tokens": 500,
  "system": "${system_prompt}",
  "messages": [
    {
      "role": "user",
      "content": "${user_message}"
    }
  ]
}

Variables: (automatically detected from ${variable_name} in body)
  system_prompt
  user_message
  api_key

Save.
```

### Step 5: Test the REST Method

```
On the HTTP Method record → click "Test"
Fill in test values:
  system_prompt: You are a helpful assistant.
  user_message: Say hello in one word.
  api_key: [your actual key]

Click "Send"
Expected response: 200 OK with JSON body containing content[0].text
```

---

## Creating a Flow Designer Action (Callable from Flow)

```
Navigate to: Flow Designer → Action Designer → New Action

Name: AI Eligibility Pre-screen
Application: ScholarHub (x_snc_scholar)
Description: Checks applicant eligibility using Claude API

INPUTS:
  Name: gwa               Type: String
  Name: family_income     Type: String
  Name: course            Type: String
  Name: school_type       Type: String
  Name: honor_rank        Type: String
  Name: scholarship_type  Type: String
  Name: gwa_requirement   Type: String
  Name: income_cap        Type: String
  Name: course_priority   Type: String

STEPS:
  Step 1: Script (Build the prompt)
    Script:
      var prompt = buildEligibilityPrompt(
        inputs.gwa, inputs.family_income, inputs.course,
        inputs.school_type, inputs.honor_rank,
        inputs.scholarship_type, inputs.gwa_requirement,
        inputs.income_cap, inputs.course_priority
      );
      outputs.user_message = prompt;

  Step 2: REST Step
    Connection: Anthropic Claude
    REST Message: claude_messages
    HTTP Method: check_eligibility
    Variables:
      system_prompt = "You are a scholarship eligibility assistant..."
      user_message = [Step 1 output: user_message]
      api_key = [use Credential lookup or system property]

  Step 3: Script (Parse response)
    Script:
      var responseBody = steps['REST Step'].responseBody;
      try {
        var parsed = JSON.parse(responseBody);
        var text = parsed.content[0].text;
        var result = JSON.parse(text);
        outputs.eligibility_status = result.status || 'not_eligible';
        outputs.matched = JSON.stringify(result.matched || []);
        outputs.missing = JSON.stringify(result.missing || []);
        outputs.note = result.note || '';
      } catch(e) {
        outputs.eligibility_status = 'error';
        outputs.matched = '[]';
        outputs.missing = '[]';
        outputs.note = 'Eligibility check unavailable. Please apply manually.';
      }

OUTPUTS:
  Name: eligibility_status  Type: String  (eligible / partial / not_eligible / error)
  Name: matched             Type: String  (JSON array string)
  Name: missing             Type: String  (JSON array string)
  Name: note                Type: String
```

---

## Calling the Action from Flow Designer

```
In Flow Designer → Add Action Step:
  Search for: "AI Eligibility Pre-screen"
  Map inputs from flow variables:
    gwa           → trigger.current.gwa
    family_income → trigger.current.family_income
    course        → trigger.current.course
    ...

Use outputs in subsequent steps:
  [Action Step].eligibility_status
  [Action Step].matched
  [Action Step].missing
```

---

## Calling from a Service Portal Widget

For real-time eligibility check (not via Flow), call the REST endpoint directly from the widget server script:

```javascript
// Widget server script
var rm = new sn_ws.RESTMessageV2('claude_messages', 'check_eligibility');
rm.setStringParameterNoEscape('system_prompt', systemPromptText);
rm.setStringParameterNoEscape('user_message', userPromptText);
rm.setStringParameterNoEscape('api_key', gs.getProperty('x_snc_scholar.claude_api_key'));

try {
    var response = rm.execute();
    var httpStatus = response.getStatusCode();
    var body = response.getBody();

    if (httpStatus == 200) {
        var parsed = JSON.parse(body);
        var resultText = parsed.content[0].text;
        var result = JSON.parse(resultText);
        data.eligibilityStatus = result.status;
        data.matched = result.matched;
        data.missing = result.missing;
        data.note = result.note;
    } else {
        data.eligibilityStatus = 'error';
        data.note = 'Service temporarily unavailable.';
    }
} catch(e) {
    data.eligibilityStatus = 'error';
    data.note = 'Could not check eligibility. You may still apply.';
    gs.log('ScholarHub AI error: ' + e.message, 'ScholarHub');
}
```

---

## Storing the API Key Securely

```
DO NOT hardcode the API key in scripts.

Option A (simplest for demo): Store as System Property
  Navigate to: sys_properties_list.do → New
  Name: x_snc_scholar.claude_api_key
  Value: sk-ant-[your key]
  Type: password2 (encrypted)

  Access in script:
  gs.getProperty('x_snc_scholar.claude_api_key')

Option B (more proper): Connection Alias Credential
  Already covered in Step 1 above.
  Reference via the REST Message configuration.
```

---

## Integration Hub Execution Logs

```
Check execution logs at:
  Navigate to: Integration Hub → Logs → REST Executions
  OR: /sn_ih_log_list.do

Useful columns: Created, Duration, Status Code, Request/Response Body
Filter by: REST Message = claude_messages

If you see 401: API key is wrong or not set in the credential
If you see 422: Request body format is invalid (check JSON)
If you see 429: Rate limited — wait and retry
If you see 500: Claude API internal error — retry after a minute
```
