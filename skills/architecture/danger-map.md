# Danger Map — ScholarHub High-Risk Areas
> Agent: tread carefully in these areas. Confirm before making changes. One mistake cascades.

---

## CRITICAL — Stop and Confirm Before Touching

### Flow Designer Application Lifecycle Flow
- **Risk:** Changing trigger condition breaks the entire workflow for all applications
- **Safe change:** Add new action steps, modify email content
- **Dangerous:** Changing trigger table, trigger condition, or deleting wait steps
- **Before changing:** Export the current flow as XML backup first
- **Rule:** Do not modify a working flow step without explicit instruction

### Integration Hub Connection Alias (Anthropic Claude API)
- **Risk:** Deleting or misconfiguring the credential locks all AI pre-screen calls
- **Safe change:** Update the API key value in the credential record
- **Dangerous:** Deleting the Connection Alias, changing the auth type
- **Rule:** Never store the API key anywhere except the ServiceNow Credential record

### Scoped Application Manifest
- **Risk:** Changing scope prefix (`x_snc_scholar`) breaks all table references
- **Rule:** Never change the application scope after tables are created

### sys_user Table
- **Risk:** Modifying system user records can break authentication
- **Rule:** Only interact with sys_user via reference fields — never modify user records directly

---

## HIGH — Extra Testing Required

### Business Rule: sn_scholar_validate_documents
- Used by every application submission — a bug blocks all applications
- Test after every change with both complete and incomplete doc sets

### Business Rule: sn_scholar_flag_non_compliant
- Changes award.status — cascading effect on scholar access and disbursement
- Test with impersonated scholar user to confirm access change

### Inbound Email Action
- Tied to production-ish email address — test changes carefully
- Parser must handle edge cases: lowercase `approve`, extra whitespace, HTML email vs plain text

### User Criteria Scripts
- A wrong criteria script locks out users from the portal
- Always test with impersonation immediately after saving

### ACL Records
- A wrong ACL can expose records to wrong roles or lock legitimate users out
- Test every ACL change with all three role impersonations

---

## MEDIUM — Test Before Committing

### Client Scripts
- `Dynamic Document Checklist` (onLoad) — fires for every application form load
- If this script errors, the entire application form breaks

### Email Templates with ${} tokens
- Unresolved tokens show as literal `${field_name}` in sent emails — embarrassing in demo
- Always send a test email before marking template done

### Scheduled Business Rules
- The auto-close scholarship BR runs daily — test by manually triggering it
- Don't leave it in a state that closes all open scholarships accidentally

---

## Danger Map: Files by Risk Level

```
CRITICAL (never touch without backup):
  Flow Designer: ScholarHub Application Lifecycle
  Integration: Anthropic Claude API Connection Alias
  Application: Scoped App Manifest

HIGH (test all 3 roles after change):
  Business Rules: validate_documents, flag_non_compliant
  Email: Inbound Email Action
  Portal: All User Criteria scripts
  ACLs: All 5 tables

MEDIUM (test affected role after change):
  Client Scripts: Dynamic Document Checklist
  Email Templates: Any with ${} tokens
  Scheduled BRs: Auto-close scholarships
```
