# UI Design System — ScholarHub Service Portal
> Agent: every widget uses this system. Professional, modern, ServiceNow-native.
> No violet. No emoji decorations. No generic Bootstrap blue. No drop shadows everywhere.

---

## Design Philosophy (5 Rules That Make It Non-Generic)

**Rule 1 — Color encodes meaning, not decoration.**
Every color choice has a reason: green = success/scholarship system, navy = authority/structure, amber = warning, red = error. Never use a color just to make something "pop."

**Rule 2 — Whitespace is the layout, not filler.**
ServiceNow's Now Platform feels clean because spacing is generous. 24px between cards, 16px internal padding minimum. If it feels "empty," that's correct.

**Rule 3 — Typography does the heavy lifting.**
Two weights only: 400 (body) and 600 (headings/labels). Size contrast (14px body vs 22px heading) creates hierarchy more powerfully than color.

**Rule 4 — Status is instant.**
A user should understand the state of any record in under 1 second. Status pills, left-border accents, and background fills communicate state — not just text.

**Rule 5 — One accent per page.**
The primary green (#1E8A63) appears on exactly one action at a time — the most important button or the active nav item. Every other element is neutral. The eye always knows where to go.

---

## Color Token System

Paste these into your widget's CSS section. These tokens reference ServiceNow's Now Platform aesthetic — deep greens, authoritative navy, clean neutrals.

```css
/* ── SCHOLARHUB DESIGN TOKENS ──────────────────────────── */
/* Primary Brand — deep forest green (ServiceNow-adjacent) */
:root {
  --sh-brand:         #1E8A63;  /* primary buttons, links, active states    */
  --sh-brand-dark:    #145A42;  /* hover on primary buttons, nav header     */
  --sh-brand-deep:    #0B3D2E;  /* page header bar, sidebar                 */
  --sh-brand-mid:     #25A87A;  /* focus rings, progress bars               */
  --sh-brand-tint:    #D4EDE5;  /* tag backgrounds, light accents           */
  --sh-brand-ghost:   #EBF7F3;  /* row hover, section backgrounds           */

  /* Secondary — ServiceNow navy (authoritative, structural)               */
  --sh-navy:          #1C3450;  /* secondary buttons, table headers         */
  --sh-navy-dark:     #0F1E2D;  /* nav sidebar active, deep headings        */
  --sh-navy-tint:     #D8E4EE;  /* info chip backgrounds                    */
  --sh-navy-ghost:    #EEF4F9;  /* secondary row hover                      */

  /* Neutral grays                                                          */
  --sh-text:          #111827;  /* primary text (headings)                  */
  --sh-text-body:     #374151;  /* body text, descriptions                  */
  --sh-text-muted:    #6B7280;  /* labels, placeholders, secondary info     */
  --sh-text-hint:     #9CA3AF;  /* helper text, disabled states             */
  --sh-border:        #E5E7EB;  /* card borders, dividers                   */
  --sh-border-mid:    #D1D5DB;  /* input borders, stronger dividers         */
  --sh-bg-page:       #F3F4F6;  /* page background                          */
  --sh-bg-surface:    #FFFFFF;  /* card, panel, modal backgrounds           */
  --sh-bg-row:        #F9FAFB;  /* alternating table rows                   */

  /* Semantic status                                                        */
  --sh-success-bg:    #ECFDF5;  --sh-success-border: #059669;  --sh-success-text: #065F46;
  --sh-warning-bg:    #FFFBEB;  --sh-warning-border: #D97706;  --sh-warning-text: #92400E;
  --sh-error-bg:      #FEF2F2;  --sh-error-border:   #DC2626;  --sh-error-text:   #991B1B;
  --sh-info-bg:       #EFF6FF;  --sh-info-border:    #2563EB;  --sh-info-text:    #1E40AF;

  /* Sizing                                                                 */
  --sh-radius-sm:    4px;
  --sh-radius-md:    6px;
  --sh-radius-lg:    10px;
  --sh-radius-pill:  999px;
  --sh-shadow-card:  0 1px 3px rgba(0,0,0,0.08), 0 1px 2px rgba(0,0,0,0.04);
  --sh-shadow-focus: 0 0 0 3px rgba(30, 138, 99, 0.2);
}
```

---

## Typography

```css
/* Base font — ServiceNow ships Source Sans Pro. Fallback to system-ui. */
body, .sp-widget { font-family: "Source Sans Pro", system-ui, sans-serif; }

.sh-page-title  { font-size: 22px; font-weight: 600; color: var(--sh-text); line-height: 1.3; }
.sh-section-title{ font-size: 17px; font-weight: 600; color: var(--sh-text); line-height: 1.4; }
.sh-card-title  { font-size: 15px; font-weight: 600; color: var(--sh-text); line-height: 1.4; }
.sh-label       { font-size: 12px; font-weight: 600; color: var(--sh-text-muted);
                  letter-spacing: 0.06em; text-transform: uppercase; }
.sh-body        { font-size: 14px; font-weight: 400; color: var(--sh-text-body); line-height: 1.6; }
.sh-caption     { font-size: 12px; font-weight: 400; color: var(--sh-text-muted); line-height: 1.5; }
.sh-mono        { font-family: "JetBrains Mono", monospace; font-size: 13px; }

/* Numbers and data — slightly tabular for alignment */
.sh-stat-number { font-size: 28px; font-weight: 600; color: var(--sh-text); letter-spacing: -0.02em; }
.sh-stat-label  { font-size: 12px; font-weight: 400; color: var(--sh-text-muted); margin-top: 2px; }
```

---

## Spacing Scale

```
4px   — icon gap, tight inline spacing
8px   — between label and value, between badge elements
12px  — between small related items
16px  — card internal padding (minimum)
20px  — standard card padding
24px  — between cards, section internal padding
32px  — between sections
48px  — between major page sections
```

---

## Component Library

### 1. Page Header Bar

```html
<div class="sh-page-header">
  <div class="sh-page-header-inner">
    <div>
      <p class="sh-label" style="margin:0 0 4px;">ScholarHub</p>
      <h1 class="sh-page-title" style="margin:0;">Scholarship catalog</h1>
    </div>
    <button class="sh-btn sh-btn-primary">Post scholarship</button>
  </div>
</div>
```
```css
.sh-page-header {
  background: var(--sh-brand-deep);
  padding: 0;
  margin: -20px -20px 24px; /* break out of SP widget padding */
}
.sh-page-header-inner {
  max-width: 1100px; margin: 0 auto; padding: 20px 24px;
  display: flex; align-items: center; justify-content: space-between;
}
.sh-page-header .sh-label { color: rgba(255,255,255,0.55); }
.sh-page-header .sh-page-title { color: #ffffff; }
```

---

### 2. Buttons

```html
<!-- Primary action (one per page) -->
<button class="sh-btn sh-btn-primary">Apply now</button>

<!-- Secondary action -->
<button class="sh-btn sh-btn-secondary">Save draft</button>

<!-- Ghost / outline -->
<button class="sh-btn sh-btn-ghost">Cancel</button>

<!-- Danger -->
<button class="sh-btn sh-btn-danger">Deny application</button>

<!-- Small (for in-table or inline actions) -->
<button class="sh-btn sh-btn-primary sh-btn-sm">View</button>
```

```css
.sh-btn {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 9px 18px; border-radius: var(--sh-radius-md);
  font-size: 14px; font-weight: 600; line-height: 1;
  border: 1.5px solid transparent; cursor: pointer;
  transition: background 0.15s, border-color 0.15s, transform 0.1s;
  text-decoration: none; white-space: nowrap;
}
.sh-btn:active { transform: scale(0.98); }

.sh-btn-primary {
  background: var(--sh-brand); color: #fff;
  border-color: var(--sh-brand);
}
.sh-btn-primary:hover { background: var(--sh-brand-dark); border-color: var(--sh-brand-dark); }
.sh-btn-primary:focus { outline: none; box-shadow: var(--sh-shadow-focus); }

.sh-btn-secondary {
  background: var(--sh-navy); color: #fff;
  border-color: var(--sh-navy);
}
.sh-btn-secondary:hover { background: var(--sh-navy-dark); border-color: var(--sh-navy-dark); }

.sh-btn-ghost {
  background: transparent; color: var(--sh-text-body);
  border-color: var(--sh-border-mid);
}
.sh-btn-ghost:hover { background: var(--sh-bg-row); }

.sh-btn-danger {
  background: transparent; color: var(--sh-error-text);
  border-color: var(--sh-error-border);
}
.sh-btn-danger:hover { background: var(--sh-error-bg); }

.sh-btn-sm { padding: 6px 12px; font-size: 13px; }
```

---

### 3. Cards

```html
<!-- Standard card -->
<div class="sh-card">
  <div class="sh-card-header">
    <h3 class="sh-card-title">SM Foundation Scholarship</h3>
    <span class="sh-badge sh-badge-success">Open</span>
  </div>
  <div class="sh-card-body">
    <p class="sh-body">Full tuition + monthly stipend for BS Information Technology students.</p>
  </div>
  <div class="sh-card-footer">
    <span class="sh-caption">Deadline: June 15, 2026</span>
    <button class="sh-btn sh-btn-primary sh-btn-sm">View details</button>
  </div>
</div>

<!-- Stat card (summary number) -->
<div class="sh-stat-card">
  <p class="sh-stat-number">24</p>
  <p class="sh-stat-label">Applications received</p>
</div>

<!-- Accent card (left border for status) -->
<div class="sh-card sh-card-accent sh-card-accent-success">
  <h3 class="sh-card-title">Award Active</h3>
  <p class="sh-body">SM Foundation Scholarship — Full Tuition</p>
</div>
```

```css
.sh-card {
  background: var(--sh-bg-surface);
  border: 1px solid var(--sh-border);
  border-radius: var(--sh-radius-lg);
  overflow: hidden;
  transition: box-shadow 0.15s;
}
.sh-card:hover { box-shadow: var(--sh-shadow-card); }

.sh-card-header {
  padding: 16px 20px 12px;
  display: flex; align-items: flex-start;
  justify-content: space-between; gap: 12px;
}
.sh-card-body  { padding: 0 20px 12px; }
.sh-card-footer {
  padding: 12px 20px;
  border-top: 1px solid var(--sh-border);
  background: var(--sh-bg-row);
  display: flex; align-items: center;
  justify-content: space-between;
}

/* Stat card */
.sh-stat-card {
  background: var(--sh-bg-surface);
  border: 1px solid var(--sh-border);
  border-radius: var(--sh-radius-lg);
  padding: 20px;
  text-align: center;
}

/* Accent border cards */
.sh-card-accent          { border-left-width: 3px; border-radius: 0 var(--sh-radius-lg) var(--sh-radius-lg) 0; }
.sh-card-accent-success  { border-left-color: var(--sh-success-border); }
.sh-card-accent-warning  { border-left-color: var(--sh-warning-border); }
.sh-card-accent-error    { border-left-color: var(--sh-error-border);   }
.sh-card-accent-info     { border-left-color: var(--sh-info-border);    }
.sh-card-accent-brand    { border-left-color: var(--sh-brand);          }
```

---

### 4. Status Badges & Pills

```html
<!-- Status: application states -->
<span class="sh-badge sh-badge-gray">Draft</span>
<span class="sh-badge sh-badge-info">Submitted</span>
<span class="sh-badge sh-badge-warning">Under review</span>
<span class="sh-badge sh-badge-success">Approved</span>
<span class="sh-badge sh-badge-error">Denied</span>

<!-- Scholarship type tags -->
<span class="sh-tag sh-tag-merit">Merit</span>
<span class="sh-tag sh-tag-need">Need-based</span>
<span class="sh-tag sh-tag-combined">Combined</span>

<!-- Dot indicator (compact inline status) -->
<span class="sh-dot sh-dot-success"></span> Active
<span class="sh-dot sh-dot-warning"></span> On hold
<span class="sh-dot sh-dot-error"></span>   Terminated
```

```css
/* Badges — filled pill for status */
.sh-badge {
  display: inline-flex; align-items: center;
  padding: 3px 10px; border-radius: var(--sh-radius-pill);
  font-size: 12px; font-weight: 600; white-space: nowrap;
}
.sh-badge-success  { background: var(--sh-success-bg); color: var(--sh-success-text); }
.sh-badge-warning  { background: var(--sh-warning-bg); color: var(--sh-warning-text); }
.sh-badge-error    { background: var(--sh-error-bg);   color: var(--sh-error-text);   }
.sh-badge-info     { background: var(--sh-info-bg);    color: var(--sh-info-text);    }
.sh-badge-gray     { background: var(--sh-bg-row);     color: var(--sh-text-muted);   }
.sh-badge-brand    { background: var(--sh-brand-tint); color: var(--sh-brand-dark);   }

/* Tags — outlined pill for categories */
.sh-tag {
  display: inline-flex; align-items: center;
  padding: 2px 8px; border-radius: var(--sh-radius-pill);
  font-size: 12px; font-weight: 600; border: 1.5px solid;
}
.sh-tag-merit    { color: var(--sh-navy);        border-color: var(--sh-navy-tint);  background: var(--sh-navy-ghost); }
.sh-tag-need     { color: var(--sh-brand-dark);  border-color: var(--sh-brand-tint); background: var(--sh-brand-ghost); }
.sh-tag-combined { color: var(--sh-warning-text); border-color: #FDE68A;             background: var(--sh-warning-bg); }

/* Status dot */
.sh-dot {
  display: inline-block; width: 8px; height: 8px;
  border-radius: 50%; margin-right: 4px; vertical-align: middle;
}
.sh-dot-success { background: var(--sh-success-border); }
.sh-dot-warning { background: var(--sh-warning-border); }
.sh-dot-error   { background: var(--sh-error-border);   }
.sh-dot-brand   { background: var(--sh-brand); }
```

---

### 5. Form Inputs

```html
<div class="sh-field">
  <label class="sh-label" for="gwa">General weighted average</label>
  <input class="sh-input" type="number" id="gwa"
         placeholder="e.g., 90.5" min="0" max="100" step="0.1" />
  <p class="sh-caption">Enter your most recent semester GWA.</p>
</div>

<div class="sh-field">
  <label class="sh-label" for="income">Annual family income (PHP)</label>
  <div class="sh-input-prefix">
    <span class="sh-input-prefix-text">₱</span>
    <input class="sh-input sh-input-has-prefix" type="number" id="income"
           placeholder="e.g., 250000" />
  </div>
</div>

<div class="sh-field">
  <label class="sh-label" for="scholarship-type">Scholarship type</label>
  <select class="sh-select" id="scholarship-type">
    <option value="">Select type</option>
    <option value="merit">Merit-based</option>
    <option value="need_based">Need-based</option>
    <option value="combined">Combined</option>
  </select>
</div>

<!-- Input with validation error -->
<div class="sh-field sh-field-error">
  <label class="sh-label" for="gwa-err">GWA</label>
  <input class="sh-input sh-input-error" type="number" value="60" />
  <p class="sh-field-error-msg">GWA does not meet the minimum requirement of 85.</p>
</div>
```

```css
.sh-field        { margin-bottom: 20px; }
.sh-field .sh-label { display: block; margin-bottom: 6px; }

.sh-input, .sh-select {
  width: 100%; padding: 9px 12px;
  border: 1.5px solid var(--sh-border-mid);
  border-radius: var(--sh-radius-md);
  font-size: 14px; color: var(--sh-text-body);
  background: var(--sh-bg-surface);
  transition: border-color 0.15s, box-shadow 0.15s;
  outline: none; -webkit-appearance: none; appearance: none;
}
.sh-input:focus, .sh-select:focus {
  border-color: var(--sh-brand);
  box-shadow: var(--sh-shadow-focus);
}
.sh-input::placeholder { color: var(--sh-text-hint); }

.sh-input-prefix   { position: relative; }
.sh-input-prefix-text {
  position: absolute; left: 12px; top: 50%;
  transform: translateY(-50%);
  font-size: 14px; color: var(--sh-text-muted);
  pointer-events: none;
}
.sh-input-has-prefix { padding-left: 26px; }

.sh-input-error    { border-color: var(--sh-error-border) !important; }
.sh-field-error-msg {
  margin: 4px 0 0; font-size: 12px;
  color: var(--sh-error-text); font-weight: 400;
}

.sh-field .sh-caption { margin: 4px 0 0; }
```

---

### 6. Data Tables

```html
<div class="sh-table-wrap">
  <table class="sh-table">
    <thead>
      <tr>
        <th>Applicant</th>
        <th>Scholarship</th>
        <th>GWA</th>
        <th>Submitted</th>
        <th>Status</th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><strong>Juan dela Cruz</strong><br><span class="sh-caption">BSIT — 2nd year</span></td>
        <td>SM Foundation</td>
        <td>90.5</td>
        <td>May 12, 2026</td>
        <td><span class="sh-badge sh-badge-warning">Under review</span></td>
        <td><a class="sh-btn sh-btn-ghost sh-btn-sm" href="#">Review</a></td>
      </tr>
      <tr class="sh-table-row-alt">
        <td><strong>Maria Santos</strong><br><span class="sh-caption">BSCS — 3rd year</span></td>
        <td>Landbank</td>
        <td>88.0</td>
        <td>May 10, 2026</td>
        <td><span class="sh-badge sh-badge-success">Approved</span></td>
        <td><a class="sh-btn sh-btn-ghost sh-btn-sm" href="#">View</a></td>
      </tr>
    </tbody>
  </table>
</div>
```

```css
.sh-table-wrap {
  border: 1px solid var(--sh-border);
  border-radius: var(--sh-radius-lg);
  overflow: hidden;
  overflow-x: auto;
}
.sh-table {
  width: 100%; border-collapse: collapse;
  font-size: 14px; color: var(--sh-text-body);
}
.sh-table thead tr {
  background: var(--sh-bg-row);
  border-bottom: 1.5px solid var(--sh-border-mid);
}
.sh-table th {
  padding: 11px 16px; text-align: left;
  font-size: 12px; font-weight: 600;
  color: var(--sh-text-muted); letter-spacing: 0.04em;
  text-transform: uppercase; white-space: nowrap;
}
.sh-table td {
  padding: 13px 16px;
  border-bottom: 1px solid var(--sh-border); vertical-align: top;
}
.sh-table tr:last-child td { border-bottom: none; }
.sh-table-row-alt { background: var(--sh-bg-row); }
.sh-table tbody tr:hover { background: var(--sh-brand-ghost); }
```

---

### 7. Alert Banners

```html
<!-- Use for global page messages, not inline validation -->
<div class="sh-alert sh-alert-success">
  Your application has been submitted. The provider will review within 5 business days.
</div>
<div class="sh-alert sh-alert-warning">
  Your compliance documents are due by June 30, 2026.
</div>
<div class="sh-alert sh-alert-error">
  Submission failed: PSA Birth Certificate and ITR are missing.
</div>
<div class="sh-alert sh-alert-info">
  This scholarship is for BSIT, BSCS, and BSECE students only.
</div>
```

```css
.sh-alert {
  padding: 12px 16px; border-radius: var(--sh-radius-md);
  font-size: 14px; line-height: 1.5; border-left-width: 3px;
  border-left-style: solid; border-top: 1px solid;
  border-right: 1px solid; border-bottom: 1px solid;
  margin-bottom: 16px;
}
.sh-alert-success { background: var(--sh-success-bg); border-color: var(--sh-success-border); color: var(--sh-success-text); }
.sh-alert-warning { background: var(--sh-warning-bg); border-color: var(--sh-warning-border); color: var(--sh-warning-text); }
.sh-alert-error   { background: var(--sh-error-bg);   border-color: var(--sh-error-border);   color: var(--sh-error-text);   }
.sh-alert-info    { background: var(--sh-info-bg);     border-color: var(--sh-info-border);    color: var(--sh-info-text);    }
```

---

### 8. Empty States

```html
<!-- Used when no records are found — never show a blank page -->
<div class="sh-empty">
  <div class="sh-empty-icon">
    <svg width="40" height="40" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="1.2">
      <path stroke-linecap="round" stroke-linejoin="round"
        d="M9 12h6m-3-3v6M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
    </svg>
  </div>
  <p class="sh-empty-title">No scholarships available</p>
  <p class="sh-empty-body">Check back soon — new scholarships are added regularly.</p>
</div>

<!-- With action -->
<div class="sh-empty">
  <p class="sh-empty-title">No applications yet</p>
  <p class="sh-empty-body">Browse the catalog to find a scholarship that fits you.</p>
  <a class="sh-btn sh-btn-primary" href="/scholarhub?id=homepage">Browse catalog</a>
</div>
```

```css
.sh-empty {
  text-align: center; padding: 48px 24px;
  border: 1px dashed var(--sh-border-mid);
  border-radius: var(--sh-radius-lg);
  background: var(--sh-bg-row);
}
.sh-empty-icon {
  width: 56px; height: 56px; margin: 0 auto 16px;
  background: var(--sh-brand-ghost);
  border-radius: 50%; display: flex;
  align-items: center; justify-content: center;
  color: var(--sh-brand);
}
.sh-empty-title {
  font-size: 16px; font-weight: 600;
  color: var(--sh-text); margin: 0 0 6px;
}
.sh-empty-body {
  font-size: 14px; color: var(--sh-text-muted);
  margin: 0 0 20px; max-width: 340px; margin-left: auto; margin-right: auto;
}
```

---

## ScholarHub-Specific Components

### Eligibility Result Card

```html
<!-- Eligible -->
<div class="sh-eligibility sh-eligibility-eligible">
  <div class="sh-eligibility-header">
    <div class="sh-eligibility-dot"></div>
    <strong>You appear eligible for this scholarship</strong>
  </div>
  <ul class="sh-eligibility-list">
    <li class="sh-elig-matched">GWA 90 meets minimum requirement of 85</li>
    <li class="sh-elig-matched">Annual income PHP 200,000 is within PHP 300,000 cap</li>
    <li class="sh-elig-matched">BSIT is a priority course</li>
  </ul>
  <p class="sh-caption">This is an automated pre-screen. The provider makes all final decisions.</p>
</div>

<!-- Partial -->
<div class="sh-eligibility sh-eligibility-partial">
  <div class="sh-eligibility-header">
    <div class="sh-eligibility-dot"></div>
    <strong>You may qualify — review the gaps below</strong>
  </div>
  <ul class="sh-eligibility-list">
    <li class="sh-elig-matched">GWA 90 meets minimum requirement of 85</li>
    <li class="sh-elig-missing">BSN is not a priority course (BSIT, BSCS, BSECE preferred)</li>
  </ul>
</div>

<!-- Not eligible -->
<div class="sh-eligibility sh-eligibility-ineligible">
  <div class="sh-eligibility-header">
    <div class="sh-eligibility-dot"></div>
    <strong>You do not meet the criteria for this scholarship</strong>
  </div>
  <ul class="sh-eligibility-list">
    <li class="sh-elig-missing">GWA 75 is below minimum requirement of 88</li>
    <li class="sh-elig-missing">Annual income PHP 600,000 exceeds PHP 300,000 cap</li>
  </ul>
</div>
```

```css
.sh-eligibility {
  border-radius: var(--sh-radius-lg);
  border: 1.5px solid; padding: 16px 20px;
  margin-bottom: 20px;
}
.sh-eligibility-eligible  { background: var(--sh-success-bg); border-color: var(--sh-success-border); }
.sh-eligibility-partial   { background: var(--sh-warning-bg); border-color: var(--sh-warning-border); }
.sh-eligibility-ineligible{ background: var(--sh-error-bg);   border-color: var(--sh-error-border);   }

.sh-eligibility-header {
  display: flex; align-items: center; gap: 10px;
  margin-bottom: 12px;
  font-size: 14px; font-weight: 600;
}
.sh-eligibility-eligible  .sh-eligibility-header { color: var(--sh-success-text); }
.sh-eligibility-partial   .sh-eligibility-header { color: var(--sh-warning-text); }
.sh-eligibility-ineligible .sh-eligibility-header { color: var(--sh-error-text); }

.sh-eligibility-dot {
  width: 10px; height: 10px; border-radius: 50%; flex-shrink: 0;
}
.sh-eligibility-eligible  .sh-eligibility-dot { background: var(--sh-success-border); }
.sh-eligibility-partial   .sh-eligibility-dot { background: var(--sh-warning-border); }
.sh-eligibility-ineligible .sh-eligibility-dot { background: var(--sh-error-border); }

.sh-eligibility-list {
  list-style: none; margin: 0 0 10px; padding: 0;
  display: flex; flex-direction: column; gap: 6px;
}
.sh-eligibility-list li {
  font-size: 13px; padding-left: 20px; position: relative;
}
.sh-eligibility-list li::before {
  content: ''; position: absolute; left: 0; top: 5px;
  width: 8px; height: 8px; border-radius: 50%;
}
.sh-elig-matched { color: var(--sh-success-text); }
.sh-elig-matched::before { background: var(--sh-success-border); }
.sh-elig-missing { color: var(--sh-error-text); }
.sh-elig-missing::before { background: var(--sh-error-border); }
```

---

### Application Status Timeline

```html
<div class="sh-timeline">
  <div class="sh-timeline-step sh-tl-done">
    <div class="sh-tl-dot"></div>
    <div class="sh-tl-content">
      <p class="sh-tl-title">Submitted</p>
      <p class="sh-caption">May 12, 2026 — 09:14 AM</p>
    </div>
  </div>
  <div class="sh-timeline-step sh-tl-active">
    <div class="sh-tl-dot"></div>
    <div class="sh-tl-content">
      <p class="sh-tl-title">Under review</p>
      <p class="sh-caption">Provider is reviewing your application</p>
    </div>
  </div>
  <div class="sh-timeline-step sh-tl-pending">
    <div class="sh-tl-dot"></div>
    <div class="sh-tl-content">
      <p class="sh-tl-title">Decision</p>
      <p class="sh-caption">Pending</p>
    </div>
  </div>
</div>
```

```css
.sh-timeline { padding-left: 8px; }
.sh-timeline-step {
  display: flex; gap: 14px;
  position: relative; padding-bottom: 20px;
}
.sh-timeline-step:not(:last-child) .sh-tl-dot::after {
  content: ''; position: absolute;
  left: 15px; top: 20px; bottom: 0;
  width: 1.5px; background: var(--sh-border);
}
.sh-tl-dot {
  width: 14px; height: 14px; border-radius: 50%;
  border: 2px solid; flex-shrink: 0; margin-top: 2px;
  position: relative; z-index: 1;
}
.sh-tl-done   .sh-tl-dot { background: var(--sh-brand);       border-color: var(--sh-brand);       }
.sh-tl-active .sh-tl-dot { background: var(--sh-warning-bg);  border-color: var(--sh-warning-border); }
.sh-tl-pending .sh-tl-dot{ background: var(--sh-bg-surface);  border-color: var(--sh-border-mid);  }
.sh-tl-title { font-size: 14px; font-weight: 600; color: var(--sh-text); margin: 0 0 2px; }
```

---

### Document Checklist

```html
<div class="sh-doc-list">
  <div class="sh-doc-item sh-doc-submitted">
    <div class="sh-doc-status-bar"></div>
    <div class="sh-doc-info">
      <p class="sh-doc-name">PSA Birth Certificate</p>
      <p class="sh-caption">Uploaded May 12</p>
    </div>
    <span class="sh-badge sh-badge-success">Submitted</span>
  </div>
  <div class="sh-doc-item sh-doc-missing">
    <div class="sh-doc-status-bar"></div>
    <div class="sh-doc-info">
      <p class="sh-doc-name">Income Tax Return</p>
      <p class="sh-caption">Required for need-based</p>
    </div>
    <span class="sh-badge sh-badge-error">Missing</span>
  </div>
</div>
```

```css
.sh-doc-list { display: flex; flex-direction: column; gap: 8px; }
.sh-doc-item {
  display: flex; align-items: center; gap: 12px;
  padding: 12px 16px; border-radius: var(--sh-radius-md);
  border: 1px solid var(--sh-border); background: var(--sh-bg-surface);
}
.sh-doc-status-bar {
  width: 3px; height: 36px; border-radius: 2px; flex-shrink: 0;
}
.sh-doc-submitted .sh-doc-status-bar { background: var(--sh-success-border); }
.sh-doc-missing   .sh-doc-status-bar { background: var(--sh-error-border);   }
.sh-doc-pending   .sh-doc-status-bar { background: var(--sh-border-mid);     }
.sh-doc-info { flex: 1; }
.sh-doc-name { font-size: 14px; font-weight: 600; color: var(--sh-text); margin: 0 0 2px; }
```

---

### Scholarship Card (Catalog)

```html
<div class="sh-schol-card">
  <div class="sh-schol-card-top">
    <div>
      <div style="display:flex; gap:8px; flex-wrap:wrap; margin-bottom:8px;">
        <span class="sh-tag sh-tag-need">Need-based</span>
        <span class="sh-badge sh-badge-success">Open</span>
      </div>
      <h3 class="sh-card-title" style="margin:0 0 4px;">SM Foundation Scholarship 2026</h3>
      <p class="sh-caption">SM Foundation</p>
    </div>
  </div>
  <div class="sh-schol-card-meta">
    <div class="sh-meta-row"><span class="sh-label">Min GWA</span><span class="sh-meta-val">92.0</span></div>
    <div class="sh-meta-row"><span class="sh-label">Max income</span><span class="sh-meta-val">₱150,000 / yr</span></div>
    <div class="sh-meta-row"><span class="sh-label">Slots</span><span class="sh-meta-val">10 remaining</span></div>
    <div class="sh-meta-row"><span class="sh-label">Deadline</span><span class="sh-meta-val sh-meta-val-warning">Jun 15, 2026</span></div>
  </div>
  <div class="sh-schol-card-footer">
    <button class="sh-btn sh-btn-ghost sh-btn-sm">Check eligibility</button>
    <button class="sh-btn sh-btn-primary sh-btn-sm">Apply now</button>
  </div>
</div>
```

```css
.sh-schol-card {
  background: var(--sh-bg-surface);
  border: 1px solid var(--sh-border);
  border-radius: var(--sh-radius-lg);
  overflow: hidden;
  display: flex; flex-direction: column;
}
.sh-schol-card:hover { box-shadow: var(--sh-shadow-card); border-color: var(--sh-brand-tint); }
.sh-schol-card-top   { padding: 20px 20px 16px; }
.sh-schol-card-meta  { padding: 0 20px 16px; display: flex; flex-direction: column; gap: 8px; }
.sh-meta-row { display: flex; justify-content: space-between; align-items: center; }
.sh-meta-val { font-size: 14px; font-weight: 600; color: var(--sh-text); }
.sh-meta-val-warning { color: var(--sh-warning-text); }
.sh-schol-card-footer {
  padding: 12px 20px;
  border-top: 1px solid var(--sh-border);
  background: var(--sh-bg-row);
  display: flex; gap: 8px; justify-content: flex-end;
}
```

---

## Layout Patterns

### Stats Row (4 across)

```html
<div class="sh-stats-row">
  <div class="sh-stat-card"><p class="sh-stat-number">12</p><p class="sh-stat-label">Open scholarships</p></div>
  <div class="sh-stat-card"><p class="sh-stat-number">47</p><p class="sh-stat-label">Applications received</p></div>
  <div class="sh-stat-card"><p class="sh-stat-number">8</p><p class="sh-stat-label">Awaiting review</p></div>
  <div class="sh-stat-card sh-stat-card-brand"><p class="sh-stat-number">23</p><p class="sh-stat-label">Scholars awarded</p></div>
</div>
```

```css
.sh-stats-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(0, 1fr));
  gap: 16px; margin-bottom: 28px;
}
.sh-stat-card-brand {
  border-left: 3px solid var(--sh-brand);
  background: var(--sh-brand-ghost);
}
.sh-stat-card-brand .sh-stat-number { color: var(--sh-brand-dark); }
```

### Two-column page layout

```css
.sh-page-layout {
  display: grid;
  grid-template-columns: 1fr 340px;
  gap: 24px; align-items: start;
}
@media (max-width: 768px) {
  .sh-page-layout { grid-template-columns: 1fr; }
}
```

### Catalog grid

```css
.sh-catalog-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
}
```

---

## Anti-Patterns — Never Do These

```
BAD: Blue for everything (generic Bootstrap look)
GOOD: Green for actions, navy for structure, gray for neutral

BAD: Box shadows on every card (heavy, outdated)
GOOD: 1px border + subtle hover shadow on interaction only

BAD: Gradient button backgrounds
GOOD: Flat solid color with slightly darker hover state

BAD: Colored page background (purple bg, dark bg)
GOOD: #F3F4F6 (near-white gray) page bg, white card surfaces

BAD: Icon + text label + emoji decoration all together
GOOD: Text label alone, OR small SVG icon alone — never both + emoji

BAD: Status shown only as text ("Approved")
GOOD: Status pill badge with background color + text

BAD: Different font sizes for every element (6 sizes in one widget)
GOOD: 3 sizes max: page title (22px), card title/body (15px/14px), caption/label (12px)

BAD: Empty div when no records exist
GOOD: .sh-empty component with a short message and optional action

BAD: Full-width primary button for every action
GOOD: Primary button only for the ONE main action. Others are ghost or secondary.

BAD: Red text error just under a label (easy to miss)
GOOD: .sh-field-error class on the whole field + red border + .sh-field-error-msg below input

BAD: Violet, purple, or indigo for ANY element (not in the ScholarHub palette)
GOOD: Stay within: green, navy, gray, amber, red, blue (info only)
```
