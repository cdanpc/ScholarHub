# Service Portal Widget Catalog — ScholarHub
> Agent: every widget ScholarHub needs. Use this as a build checklist and data source reference.

---

## Student View Widgets

### W1: Scholarship Catalog
```
Widget ID:      scholar-catalog
Page:           homepage
Data source:    sn_scholar_scholarship WHERE status=open
Key data:       name, type, gwa_requirement, income_cap, deadline, slots_remaining
Actions:        "View Details" → /scholarhub?id=scholarship&sys_id=[id]
Empty state:    "No scholarships are currently open. Check back soon."
Role check:     All authenticated users
```

### W2: Scholarship Detail + AI Pre-screen
```
Widget ID:      scholar-detail
Page:           scholarship (pass sys_id parameter)
Data source:    sn_scholar_scholarship by sys_id
Key data:       all fields + benefits_description + required_documents
AI section:     Form: gwa, family_income, course, school_type → call Integration Hub
AI result:      Color card: green/yellow/red with matched[] and missing[] lists
Actions:        "Apply Now" → /scholarhub?id=apply&scholarship=[id]
Role check:     sn_scholar.applicant (for Apply button and AI check)
```

### W3: Application Form
```
Widget ID:      scholar-apply
Page:           apply (pass scholarship parameter)
Data source:    sn_scholar_scholarship by id (to get requirements)
Form fields:    gwa, family_income, course, year_level, school_type, honor_rank, essay
Dynamic:        Client Script shows/hides income fields based on scholarship type
Doc checklist:  Generated from scholarship.required_documents (dynamic checkboxes)
Submit:         Creates sn_scholar_application record → triggers BR + Flow
Role check:     sn_scholar.applicant only
```

### W4: My Applications Tracker
```
Widget ID:      scholar-my-apps
Page:           my-apps
Data source:    sn_scholar_application WHERE applicant = current_user
Columns:        number, scholarship.name, status, submission_date, review_notes
Status display: Color-coded pill (draft=gray, submitted=blue, under_review=yellow, approved=green, denied=red)
Empty state:    "You have not submitted any applications yet."
Role check:     sn_scholar.applicant only
```

### W5: My Awards
```
Widget ID:      scholar-my-awards
Page:           my-awards
Data source:    sn_scholar_award WHERE scholar = current_user AND status != terminated
Columns:        scholarship.name, coverage_type, stipend_amount, start_date, end_date, status, disbursement_status
Empty state:    "You have no active scholarship awards."
Role check:     sn_scholar.applicant only
```

### W6: Compliance Submit Form
```
Widget ID:      scholar-compliance-submit
Page:           compliance
Data source:    sn_scholar_award WHERE scholar = current_user AND status = active
Form fields:    academic_year, semester, gwa_this_semester, units_enrolled
File uploads:   enrollment_cert (attachment), grade_report (attachment)
Submit:         Creates sn_scholar_compliance record
Role check:     sn_scholar.applicant + has active award (User Criteria script)
```

---

## Provider View Widgets

### W7: My Scholarships
```
Widget ID:      scholar-my-scholarships
Page:           my-schemas
Data source:    sn_scholar_scholarship WHERE provider = current_user
Columns:        name, type, slots_available, slots_remaining, deadline, status
Actions:        "View Applications" → /scholarhub?id=review-queue&scholarship=[id]
                "Post New" → opens post scholarship form
Role check:     sn_scholar.provider only
```

### W8: Post Scholarship Form
```
Widget ID:      scholar-post-scholarship
Page:           my-schemas (inline or modal)
Form fields:    name, scholarship_type, gwa_requirement, income_cap, course_priority,
                school_type_preference, slots_available, application_deadline,
                benefits_description, required_documents (multi-select), status
Submit:         Creates sn_scholar_scholarship record with provider = current_user
Role check:     sn_scholar.provider only
```

### W9: Application Review Queue
```
Widget ID:      scholar-review-queue
Page:           applications (pass scholarship parameter optional)
Data source:    sn_scholar_application WHERE scholarship.provider = current_user
                AND status != draft
Columns:        number, applicant.name, scholarship.name, status, submission_date
Filter:         By status (submitted, under_review), by scholarship
Click:          → /scholarhub?id=review&sys_id=[application-id]
Role check:     sn_scholar.provider only
```

### W10: Application Review Detail
```
Widget ID:      scholar-review-detail
Page:           review (pass sys_id parameter)
Data source:    sn_scholar_application by sys_id (with full applicant data)
                sn_scholar_document WHERE application = sys_id
Sections:       Applicant profile | AI pre-screen result | Documents list with download links
Actions:        Approve button | Deny button (prompts for reason) | Request More Docs button
                (These call UI Actions via REST or sp.GlideAjax)
Role check:     sn_scholar.provider + is provider of this scholarship
```

### W11: Scholar Roster
```
Widget ID:      scholar-roster
Page:           scholars
Data source:    sn_scholar_award WHERE scholarship.provider = current_user AND status = active
Columns:        scholar.name, scholarship.name, coverage_type, disbursement_status,
                latest_compliance_status
Actions:        "Release Disbursement" button | "View Compliance" link
Role check:     sn_scholar.provider only
```
