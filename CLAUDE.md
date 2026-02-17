# CLAUDE.md — Host Matching Tool

## Project Overview

A single-page web application for **Not Impossible** that enables teachers and educators to nominate students for one-hour, one-on-one microplacement opportunities with vetted professional hosts. The tool targets students who lack family connections for career opportunities.

**Live usage**: Teachers open `index.html` in a browser, browse/filter 400+ professional hosts across 11 subject areas, nominate students by email, and submit nominations to Airtable via a Zapier webhook.

## Repository Structure

```
host-matching/
├── index.html    # Entire application (HTML + CSS + JS in one file)
├── README.md     # One-line project description
└── CLAUDE.md     # This file
```

This is a **zero-dependency, single-file application**. There is no `package.json`, no build system, no bundler, no test framework, and no CI/CD pipeline. The file can be opened directly in a browser.

## Architecture (index.html)

The ~955-line `index.html` is organized into three sections:

### 1. CSS (lines 8–404)
- Embedded in a `<style>` tag
- CSS custom properties for theming (prefix `--ni-*`)
- Brand colors: `--ni-coral: #FF6B5B` (primary), `--ni-navy: #1A2744` (secondary)
- Font: DM Sans (body), Fraunces (headings) via Google Fonts
- Mobile-first responsive design with breakpoint at 640px
- Print styles included

### 2. HTML (lines 407–438)
- Semantic structure with a sticky header, main content area, fixed export bar, and toast container
- 4-step user flow:
  1. Teacher enters name and school
  2. Browse/filter hosts, nominate students by email
  3. Review and submit nominations
  4. Confirmation message (shown after submission)
- Key element IDs: `teacherName`, `teacherSchool`, `subjectFilter`, `searchInput`, `hostList`, `exportBar`, `submitBtn`, `toast`

### 3. JavaScript (lines 440–953)
- **Host data array** (lines 442–696): 400+ host objects embedded directly in JS
- **State management** (lines 698–717): `studentAssignments` object persisted to `localStorage` under key `ni-host-assignments`
- **Rendering** (lines 731–821): `render()` filters and rebuilds the DOM on every state change
- **Event handling** (lines 823–871): Delegated click/keydown handlers on `#hostList`
- **Airtable integration** (lines 874–939): `submitToAirtable()` sends individual POSTs to Zapier webhook
- **Toast notifications** (lines 944–949): `showToast(msg)` with 2-second auto-dismiss

## Key Data Structures

**Host object:**
```js
{
  name: string,        // "Emma Edwards"
  company: string,     // "Yoyo Design"
  desc: string,        // Role description
  subject: string,     // Category (e.g. "Business Studies & Economics")
  linkedin?: boolean,  // Has LinkedIn profile
  website?: string     // Company domain (no protocol)
}
```

**State:**
```js
studentAssignments = {
  "Name__Company": ["student1@school.ac.uk", "student2@school.ac.uk"]
}
```

Host keys are generated via `hostKey(host)` → `"${name}__${company}"`.

## Subject Categories

Business Studies & Economics, English Language & Literature, Computer Science & IT, Engineering & DT, Biology, Geography & Environmental Science, Art & Design, Media Studies, Psychology, Sociology, Law

## External Integration

- **Zapier webhook**: `https://hooks.zapier.com/hooks/catch/15285814/uc41hx8/`
- Sends one POST per nomination with fields: `host_name`, `company`, `subject`, `student_email`, `submitted_by`, `school`
- Uses `mode: 'no-cors'` for cross-origin requests
- Connects to an Airtable backend (not directly accessed)

## Development Notes

### Running Locally
Open `index.html` in any modern browser. No server or build step required.

### No Tooling
- No package manager, linter, formatter, or test runner
- No TypeScript — plain JavaScript only
- No CI/CD or automated checks

### Conventions
- All code lives in a single file; avoid splitting unless there's a strong reason
- CSS uses custom properties (`--ni-*` prefix) for all theme values
- Event delegation pattern on `#hostList` for dynamically created elements
- State changes follow: user action → update `studentAssignments` → `saveState()` → `render()`
- Email validation is minimal (checks for `@` only)
- `CSS.escape()` is used when building attribute selectors from host keys

### When Modifying Hosts
- Host data is a flat array starting at line 442, grouped by comment headers (`// BUSINESS STUDIES & ECONOMICS`, etc.)
- Each host must have `name`, `company`, `desc`, and `subject`; `linkedin` and `website` are optional
- The subject filter dropdown auto-populates from unique `subject` values in the array
- Adding a new subject category requires no code changes beyond adding hosts with that subject

### Sensitive Data
- The Zapier webhook URL is hardcoded in the source (line 874)
- No credentials, API keys, or secrets beyond the webhook URL
- Teacher/student data is stored only in browser localStorage and sent to Zapier
