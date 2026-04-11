# Reference Credentials — ScholarHub
> Agent: every external service the project needs. Never hardcode these values in scripts.

---

## Credential Map

| Service | What It Needs | Where Stored | How to Verify |
|---------|---------------|--------------|---------------|
| Anthropic Claude API | API Key (`sk-ant-...`) | ServiceNow Connection Alias: "Anthropic Claude API" | Test REST Message in Integration Hub |
| GitHub | Personal Access Token (PAT) | ServiceNow Source Control credential | Studio → Source Control → Show Changes |
| ServiceNow PDI | Instance URL + admin credentials | Team's shared document | Visit instance URL in browser |
| Test email (outbound) | SMTP configured in ServiceNow | System Email Properties | Send test notification |
| Test email (inbound) | Email address configured | Inbound Email Accounts | Reply to a test notification |

---

## Anthropic Claude API

```
Endpoint:    https://api.anthropic.com/v1/messages
Model:       claude-haiku-4-5-20251001
Auth type:   Custom header: x-api-key: [API_KEY]
Also needed: anthropic-version: 2023-06-01
Max tokens:  500 (sufficient for eligibility check)
Rate limit:  ~50 requests/min on free tier (more than enough for demo)
```

**Where to get the API key:**
1. Go to: https://console.anthropic.com/
2. Settings → API Keys → Create Key
3. Copy the key (shown once only)
4. Store in ServiceNow: Integration Hub → Connections → New Connection → Basic Auth
   - Name: "Anthropic Claude API"
   - Authentication type: Custom
   - Header name: x-api-key
   - Header value: [your API key]

**NEVER:** Put the API key directly in a Script Include, Business Rule, or Flow Action. Always reference the Connection Alias.

---

## GitHub PAT (Personal Access Token)

```
Required scopes: repo, workflow
Expiry: Set to 90 days minimum (beyond hackathon date)
```

**Where to get the PAT:**
1. GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)
2. Generate new token → check: repo, workflow
3. Copy immediately (shown once)
4. Store in ServiceNow Studio when linking repository

---

## ServiceNow PDI

```
Instance URL format: https://[dev-instance-id].service-now.com
Admin username:      admin
Admin password:      [set during PDI creation]

Important: PDI hibernates after 10 days of inactivity.
To prevent: Log in and save any record at least every 9 days.
To wake up: Visit the instance URL and click "Wake up" if hibernated.
```

---

## Test User Accounts (Create on Day 1)

| Account | Password | Role | Purpose |
|---------|----------|------|---------|
| applicant@scholarhub.test | ScholarTest123! | sn_scholar.applicant | Test student flow |
| provider@scholarhub.test | ScholarTest123! | sn_scholar.provider | Test provider flow |
| admin@scholarhub.test | ScholarTest123! | sn_scholar.admin | Test admin access |

**Creating test users:**
1. Navigate to: sys_user_list.do
2. New → fill in name, email, username
3. Set password
4. Roles tab → add the appropriate sn_scholar role
5. Save
