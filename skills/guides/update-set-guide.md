# Update Set Guide — ServiceNow Deployment
> Agent: Update Sets capture all customizations for export and sharing. Required for hackathon submission.

---

## What an Update Set Captures

An Update Set records every configuration change made while it is the "current" update set:
- Table definitions and fields
- Business Rules, Client Scripts, UI Actions, Script Includes
- Flow Designer flows
- Service Portal portals, pages, widgets
- Email notifications
- ACL records
- System Properties

**Does NOT capture:** Actual data records (GlideRecord inserts), user accounts, scheduled job run history.

---

## Setting Up the Update Set (Day 1)

```
Navigate to: All Applications → System Update Sets → Local Update Sets → New

Name: ScholarHub Development
Description: ScholarHub Scholarship Lifecycle Management System v1.0
Release date: [hackathon presentation date]
State: In Progress

Click "Make Current" — this makes it the active update set.
All changes from now on will be captured here.
```

**Verify it's active:**
The current update set name appears in the bottom right corner of the ServiceNow UI.

---

## Checking What's in Your Update Set

```
Navigate to: Local Update Sets → ScholarHub Development → Customer Updates tab

This shows every XML record captured.
If a change isn't showing here, you may be working in the wrong scope or update set.
```

---

## Exporting the Update Set (Day 7)

```
1. Navigate to: Local Update Sets → ScholarHub Development
2. Change State to "Complete"
3. Click "Export to XML"
4. Save file as: ScholarHub_v1.0_[YYYYMMDD].xml
5. Upload to GitHub repo or share with team
```

---

## Merging Multiple Update Sets (if team created separate ones)

```
Navigate to: Local Update Sets → Merge Update Sets

Select all ScholarHub update sets to merge.
Name the merged set: ScholarHub_FINAL_v1.0
Export the merged set.
```

---

## Update Set vs. GitHub Source Control

Both are used simultaneously in ScholarHub:

| | Update Set | GitHub (Studio Source Control) |
|-|------------|-------------------------------|
| Purpose | Export/import customizations | Version history, team collaboration |
| Format | XML file | XML files in git repo |
| When used | End of sprint delivery | Every commit during development |
| Who uses it | Judges to import into their instance | Team members to sync work |

**For the hackathon:** Export the Update Set XML as the formal deliverable. GitHub is for source history.

---

## If Something Is Missing from the Update Set

```
Symptom: You made changes but they don't appear in Customer Updates.

Causes:
  1. You were in the wrong update set — check bottom right of UI
  2. You edited a record in a different application scope
  3. The record was created before you set the current update set

Fix:
  Navigate to the record → right-click header → "Add to Update Set"
  Select: ScholarHub Development
  Click OK
```

---

## Importing an Update Set (for judges / testing on another instance)

```
On the target instance:
1. Navigate to: System Update Sets → Retrieved Update Sets → Import Update Set from XML
2. Upload the ScholarHub_v1.0_[date].xml file
3. Open the retrieved update set
4. Click "Preview Update Set" (checks for conflicts)
5. If no errors: click "Commit Update Set"
6. All ScholarHub tables, scripts, flows, and portal are now on that instance
```
