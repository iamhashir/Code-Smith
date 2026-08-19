# Code-Smith — Student Fee & Report Management Prototype

Code-Smith is a Next.js application for recording student fee information, tracking monthly payment status, searching saved records, and exporting selected records as **PDF, XLSX, or CSV** reports.

This repository started from a Next.js template, but the actual application contains authentication, structured student data entry, fee calculations, searchable records, and server-side document generation.

> **Stack:** Next.js 14 · React 18 · Firebase Authentication · React Hook Form · React PDF · SheetJS/XLSX · Framer Motion · Tailwind CSS

## What the application does

### Authentication

The app supports Firebase-based identity flows through a shared `AuthContext`:

- Google sign-in
- email/password registration
- email/password sign-in
- sign-out
- route guarding for data-entry/report screens

### Student data entry

Authenticated users can create records containing:

- student name
- intake
- course
- yearly fee amount
- a configurable list of monthly payments
- per-month **Paid / Unpaid** status
- arbitrary additional key/value fields when the record does not use the monthly-fee model

Courses currently represented in the form include Psychology, Creative Computing, and Business Management.

### Record management

The reporting UI can:

- list saved student records
- search by student name or intake
- inspect an individual record in a modal
- calculate total paid fees
- calculate remaining yearly balance
- delete records
- generate downloadable reports

### Document generation

`pages/api/generate-pdf.js` accepts a selected record and generates one of three formats:

```text
PDF   → @react-pdf/renderer
XLSX  → SheetJS / xlsx
CSV   → generated server-side text export
```

For fee records, exports include monthly fee rows, payment status, total paid amount, remaining due, and course information.

## Current architecture

```text
app/
├─ page.js                         marketing / product landing page
├─ login/                          authentication UI
├─ Create/
│  ├─ page.jsx                     student + fee data entry
│  └─ firebase.jsx                 Firebase client configuration
├─ upload/
│  ├─ page.js                      upload/data workflow
│  └─ Reports/page.js              search, inspect, delete and export records
├─ payment/                        pricing/payment-facing UI
└─ components/
   ├─ AuthContext.js               auth + application record state
   ├─ Navbar.js
   └─ ...

pages/api/
├─ generate-pdf.js                 PDF / XLSX / CSV document generation
├─ bills.js                        experimental bill API
├─ insert_user.js                  experimental user API
└─ upload.js                       upload endpoint prototype
```

## Data flow

The active student-record path is intentionally simple:

```text
Firebase Authentication
        ↓
Authenticated client
        ↓
Create record
        ↓
AuthContext
        ↓
localStorage (`allFormData`)
        ↓
Reports screen
        ↓
select / search / calculate
        ↓
POST /api/generate-pdf
        ↓
PDF / XLSX / CSV download
```

### Important implementation note

This is a **prototype**, not a production multi-user database architecture.

Firebase is currently used for authentication, while the student records themselves are persisted in browser `localStorage` through `AuthContext`. The `bills` API also uses an in-memory demonstration array rather than durable storage.

That distinction matters: the project demonstrates the workflow and export pipeline, but moving it to production would require moving application records into a durable backend/database and enforcing authorization server-side.

## Fee model

A monthly-fee record is stored approximately as:

```js
{
  studentName,
  intake,
  course,
  yearlyAmount,
  monthlyFees: [
    { name: "January", fee: "...", status: "Paid" },
    { name: "February", fee: "...", status: "Unpaid" }
  ]
}
```

The reports screen derives:

```text
total paid = sum(month.fee where status === "Paid")
remaining due = yearly amount - total paid
```

The same values are reused when generating downloadable documents.

## Why this project is useful

The interesting part is the complete workflow rather than any one screen:

```text
identity → structured data entry → persistence → search → calculations → export
```

It combines client state, authentication, dynamic forms, business calculations, and server-generated files in one application.

It also exposes a useful engineering lesson: a functional prototype can use browser persistence to validate a workflow quickly, while productionization requires a different persistence and authorization boundary.

## Running locally

```bash
npm install
npm run dev
```

Open `http://localhost:3000`.

Production build:

```bash
npm run build
npm run start
```

## Main dependencies

| Area | Technology |
|---|---|
| Framework | Next.js 14, React 18 |
| Authentication | Firebase Auth |
| Forms | React Hook Form, Formik, Yup |
| PDF generation | `@react-pdf/renderer`, jsPDF |
| Spreadsheet export | `xlsx` |
| Downloads | `file-saver` |
| UI | Tailwind CSS, Framer Motion, Heroicons/Lucide |
| 3D/visual experiments | React Three Fiber, Drei, Three.js |

## Production improvements

The next engineering steps would be:

1. move student and payment records from `localStorage` to a durable database
2. associate every record with an authenticated user/organization
3. enforce authorization on server-side report endpoints
4. consolidate the mixed App Router / Pages Router API structure
5. remove unused/experimental dependencies and endpoints
6. add schema validation around report payloads
7. add automated tests for fee calculations and export generation

---

**Status:** functional prototype / portfolio project. The README intentionally describes the current implementation rather than presenting prototype storage as a production backend.
