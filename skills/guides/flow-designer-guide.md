# Flow Designer Guide — ServiceNow
> Agent: the Application Lifecycle Flow is the heart of ScholarHub. Build it carefully.

---

## Flow Designer Concepts

| Term | Definition |
|------|-----------|
| Flow | A sequence of automated steps triggered by an event |
| Trigger | What starts the flow (record created, record updated, scheduled) |
| Action | A single step in the flow (send email, update record, call subflow) |
| Flow Logic | Conditional branching (If/Else), loops (For Each), wait conditions |
| Subflow | A reusable flow called from another flow |
| Variable | Data passed between steps |

---

## ScholarHub Application Lifecycle Flow

### Trigger Configuration
```
Type:      Record-Based
Table:     sn_scholar_application
Trigger:   Created
Condition: status is submitted
```

### Flow Variables (set at trigger)
```
applicationSysId  → trigger.current.sys_id
scholarshipSysId  → trigger.current.scholarship
applicantSysId    → trigger.current.applicant
applicantEmail    → trigger.current.applicant.email
providerSysId     → trigger.current.scholarship.provider
providerEmail     → trigger.current.scholarship.provider.email
```

### Step-by-Step Flow Structure

```
[TRIGGER] Application Created where status = submitted
    |
    v
[ACTION 1] Send Email: "New Application Received"
    To: {{providerEmail}}
    Template: New Application Received
    |
    v
[ACTION 2] Update Record: sn_scholar_application
    Set: status = under_review
    |
    v
[FLOW LOGIC] Wait for Condition (with 7-day timeout)
    Condition: approval_action IS NOT EMPTY
    |
    v
[FLOW LOGIC] If/Else on approval_action value
    |
    ├── IF approval_action == 'request_docs'
    │       [ACTION] Send Email: "Request for More Documents"
    │           To: {{applicantEmail}}
    │           Template: Request for More Documents
    │       [ACTION] Update Record: status = draft, approval_action = ''
    │       [FLOW LOGIC] Wait for resubmission (status back to submitted)
    │       [Go back to ACTION 1]
    │
    ├── IF approval_action == 'deny'
    │       [ACTION] Send Email: "Application Denied"
    │           To: {{applicantEmail}}
    │           Template: Application Denied
    │           Include: review_notes as denial reason
    │       [END]
    │
    └── IF approval_action == 'approve'
            [ACTION] Call Subflow: "Create Award Record"
                Input: applicationSysId, scholarshipSysId, applicantSysId
            [ACTION] Send Email: "Application Approved"
                To: {{applicantEmail}}
                Template: Application Approved
            [ACTION] Update Scholarship: slots_remaining - 1
            [END]
```

### Subflow: Create Award Record

```
[INPUT] applicationSysId, scholarshipSysId, applicantSysId

[ACTION 1] Lookup Record: sn_scholar_scholarship
    Where: sys_id = scholarshipSysId
    Output: scholarshipRecord

[ACTION 2] Create Record: sn_scholar_award
    Fields:
        application = applicationSysId
        scholar = applicantSysId
        scholarship = scholarshipSysId
        coverage_type = full_tuition
        status = active
        disbursement_status = pending
        start_date = [current date]

[OUTPUT] awardSysId
```

---

## Adding Steps in Flow Designer UI

```
1. Open Flow Designer (navigate to /flow-designer)
2. Find or create your flow
3. Click "+" to add a step
4. Choose step type:
   - Action → search for ServiceNow built-in actions
   - Flow Logic → If, For Each, Wait
   - Subflow → call another flow
5. Configure the step
6. Connect variables between steps using the data picker (drag icon)
```

---

## Wait Condition Configuration

**Critical: always set a timeout on wait conditions or the flow stalls forever.**

```
Wait type: Wait for Condition
Table: sn_scholar_application
Record: [flow variable: applicationSysId]
Condition: approval_action is not empty
Max wait time: 7 days (for demo, set to 1 hour)
What to do when timeout: Continue (let next step handle it)
```

---

## Sending Email from Flow

```
Action: Send Email
Configuration:
  Target record: sn_scholar_application (the current application)
  Notification: [select your email notification template]
  OR:
  To: [enter email or flow variable]
  Subject: [subject text]
  Body: [HTML body or use template]
```

---

## Debugging Flows

```
Flow Designer → Your Flow → Execution Details tab
Shows: every execution, each step result, variable values, errors

Common issues:
- Wait condition never fires: check that approval_action field is being set by UI Action
- Email not sent: check notification template is active and recipients are correct
- Subflow fails: check input variables are correctly mapped
- Trigger not firing: verify the trigger condition matches exactly (status = 'submitted' not 'Submitted')
```

---

## Flow Variables vs. Data Pill Picker

When referencing data from a previous step, use the **data pill picker** (the icon that looks like a target):
- Click the field input
- Click the data pill icon
- Navigate: Trigger → Current Record → [field name]

Do not hardcode sys_ids or values in flow steps — always use data pills.

---

## Important Limitations

- **Flow Designer** cannot directly call a Script Include or run arbitrary JavaScript. For complex logic, use a **Script Action** (available in Flow Designer Actions).
- **Subflows** must be in the same application scope to be callable without cross-scope permissions.
- **Wait conditions** poll every 30 seconds by default. For demo, this is fine. For production, configure appropriately.
- **Email notifications** sent via Flow must reference a Notification record — they are not sent inline.
