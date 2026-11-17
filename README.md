Students Mark Register — Complete Documentation
1. Overview
App Name: Students Mark Register
School: ST. JOSEPH'S INTERNATIONAL (CBSE) SCHOOL — Krishnagiri
Purpose: A mobile-responsive web app for managing student marks across Grades 1–10 for PA (Periodic Assessment) and Term Examinations.
Stack: Single-file HTML + CSS + vanilla JavaScript (no build step, no backend).
Persistence: Browser localStorage (data survives refresh, close, and reopen).
External libraries (CDN):

xlsx@0.18.5 — Excel import/export

jspdf@2.5.1 + jspdf-autotable@3.8.2 — PDF export

File structure:

text
index.html  (the entire app)
sjik.jpg    (default school logo — optional, place next to index.html)
2. Core Concepts
2.1 Grades
Grades 1 through 10. Each grade has its own independent list of students, marks, and results. Switching grades does not carry data across.

2.2 Student Mediums
Every student belongs to one of two mediums:

Medium	Core subjects required to pass for rank eligibility
Tamil Medium	Tamil, English, Math, Science, Social
Hindi Medium	Hindi, English, Math, Science, Social
Computer and GK are displayed on results but never counted for Grand Total, Grade, or Rank.

2.3 Subjects
Eight subjects, always in this fixed order:
Tamil, English, Math, Science, Social, Computer, Hindi, GK

2.4 Exams
Four exams, grouped into two types:

Key	Name	Type
pa1	PA I	PA
pa2	PA II	PA
term1	Term I	Term
term2	Term II	Term
PA and Term have independent mark schemes, pass marks, and grade bands.

2.5 Mark Scheme
Exam type	Theory Max	Internal Max	Subject Total
PA	40	10	50
Term	80	20	100
Grand Total = sum of subject totals for the 5 core subjects only.

PA: max 250

Term: max 500

3. Business Rules
3.1 Pass / Fail
Pass mark is compared only with the Theory mark — never with Theory + Internal.

Example (pass mark = 15): theory = 14.9 → FAIL; theory = 17 → PASS.

A student failing any core subject for their medium is not eligible for rank, but still receives Grand Total and Overall Grade.

3.2 Absent (AB)
Each subject has an AB checkbox.

When ticked:

Theory and internal inputs are cleared and disabled.

Subject total and grade display as AB.

The subject is treated as 0 marks and FAIL for all calculations.

Absent subjects count toward the statistics "Absent" column.

3.3 Grade Calculation
Subject Grade — from subject total, using the Subject Grade bands.
Overall Grade — computed from the average of the 5 core subjects:

text
average = Grand Total ÷ 5
rounded = round(average)      // ≥ 0.5 rounds up; otherwise rounds down
grade   = lookup(rounded in Subject Grade bands)
Examples:

Grand Total	÷5	Rounded	Grade
250	50	50	A1
200	40	40	A2
189	37.8	38	B1
186	37.2	37	B1
175	35	35	B1
Note: The overall grade uses the same bands as the subject grades — there is no separate overall-grade table.

3.4 Ranking (Dense / Competition Ranking)
Ranking is dense: same total marks share the same rank, and the next distinct total gets the next rank (no gaps).

Example:

Student	Grand Total	Rank
A	250	1
B	250	1
C	210	2
D	200	3
E	250	1
Only rank-eligible students participate in the ranking.

Non-eligible students show "–" in the Rank column.

3.5 Rank Eligibility
A student is eligible if all core subjects pass (theory ≥ pass mark and not absent) for their medium.

One core subject fail/absent ⇒ not eligible.

4. Tabs & Screens
4.1 Students Tab
Grade selector at the top.

Count badge showing total students in the selected grade.

Add Student card:

Name field (Enter key also submits)

Medium dropdown (Tamil / Hindi)

Add button

Rejects duplicates within the same grade.

Bulk Import card:

Accepts .xlsx, .xls, .csv

Expected columns: A = Name, B = Medium

Leading S.No numeric column is auto-detected and skipped

Header rows (Name / Student / S.No) are skipped

Medium column matched case-insensitively — anything starting with h → Hindi, t → Tamil

Duplicate names are skipped

Download Template button (CSV)

Student List:

Each row shows: index, name, medium badge

✏️ Edit button → inline edit form (name + medium) with Save and Cancel

Remove button → confirmation prompt, removes student and all their marks

4.2 Marks Entry Tab
Grade + Exam selectors.

Filters: student name search, medium.

Count indicator showing Showing X of Y students.

Table with a row per student:

Sticky columns: #, Student Name

One column per subject. Each cell contains:

AB checkbox

Theory input (T)

Internal input (I)

Total (auto-computed)

Subject Grade (auto-computed)

Fail highlight: pink background when theory < pass mark

Absent highlight: amber background when AB is ticked

Max-limit guard: typing a value above the max (theory max or internal max) is rejected with a red shake and clamped to max

Enter key moves to the next visible input

4.3 Results Tab
Grade + Exam selectors.

Filters: name, medium, status (rank eligible / not eligible).

Export bar: Excel + PDF buttons.

Column visibility panel: toggle each subject (as a group of T/I/Tot/Gr) plus Theory Total, Internal Total, Grand Total, Grade, Rank.

Overall Result table:

Sticky columns: #, Student

Per subject: T, I, Tot, Gr (4 columns each)

Theory Total, Internal Total (sums across 5 core subjects)

Grand Total, Grade, Rank

Failed cells highlighted pink; absent cells highlighted amber

Rank cell shows "–" for non-eligible students

Subject Statistics table (auto-updates with filters):

Columns: Subject, Total, Appeared, Pass, Fail, Absent, Pass %, Fail %

Pass % and Fail % calculated against Appeared students only

4.4 Settings Tab
School Logo card: upload custom logo, reset to default. Logo stored in localStorage and used in header + PDF exports.

Exam Settings card:

PA / Term toggle

Theory Max, Internal Max, Pass Mark — all editable

Live updates to Marks Entry and Results when the settings belong to the current exam type

Subject & Overall Grade card:

Add / remove / edit grade band rows (min, max, grade label)

Bands are shared between subject grades and overall grades

Live updates to Marks Entry and Results

Data card:

Backup (JSON download)

Restore (JSON upload)

Clear all data (double confirmation)

5. Data Model
Stored under the localStorage key smr_v6:

json
{
  "v": 6,
  "settings": {
    "pa": {
      "theoryMax": 40,
      "internalMax": 10,
      "passMark": 15,
      "subjectGrades": [
        { "min": 41, "max": 50, "grade": "A1" },
        { "min": 36, "max": 40, "grade": "A2" }
        // ...
      ]
    },
    "term": {
      "theoryMax": 80,
      "internalMax": 20,
      "passMark": 26,
      "subjectGrades": [ /* ... */ ]
    }
  },
  "students": {
    "1": [
      { "id": "s...", "name": "Aravind Patel N", "medium": "tamil" }
    ]
  },
  "marks": {
    "1": {
      "pa1": {
        "s...": {
          "tamil":   { "theory": "22", "internal": "10", "absent": false },
          "english": { "theory": "27", "internal": "10", "absent": false }
          // ...
        }
      }
    }
  }
}
The logo is stored separately under the key smr_logo_v1 as a data URL.

6. Exports
6.1 Excel Export
Respects current filters (name / medium / status).

Respects column visibility toggles.

Structure:

Merged title rows: school name, exam title, timestamp

Two header rows:

Row 1: group headers (subject names span T/I/Total/Grade)

Row 2: sub-headers

Data rows with S.No, Student, Medium, then subject columns and totals

Subject Statistics section appended below with its own title row

Column widths tuned per column type.

6.2 PDF Export
Landscape A4.

School logo embedded at top-left (if custom logo is uploaded).

Headers: school name, exam title, timestamp, divider line.

Main table with two-row headers, color-coded cells:

Amber = absent

Pink = failed (theory/total/grade cells)

Footer: page number (right), school name (left).

Subject Statistics table appended after the main table with green headers.

Absent counts highlighted in amber within the statistics.

7. Filters & View Options
7.1 Filters (Marks Entry & Results)
Name — case-insensitive substring search.

Medium — All / Tamil / Hindi.

Status (Results only) — All / Rank Eligible / Not Eligible.

Filters affect:

Rows shown in the table

Rows counted in the "Showing X of Y" indicator

Rows counted in the Subject Statistics table

Rows included in exports

Filters are independent per tab for name/medium; status filter is Results-only.

7.2 Column Visibility (Results only)
Each subject toggles as a group (T, I, Tot, Gr).

Individual toggles for Theory Total, Internal Total, Grand Total, Grade, Rank.

S.No and Student are always visible.

Toggles affect both the on-screen table and exports.

Show All button resets all toggles to visible.

8. UI/UX Notes
Fully mobile-responsive (single-column layouts collapse gracefully).

Sticky tab bar at top; sticky table headers and left columns on wide tables.

Color coding:

Red/pink — failed subject

Amber — absent

Green — rank eligible / pass stats

Blue — grand total / overall grade context

Toast-style inline messages for add / import / logo updates.

Confirmation prompts for destructive actions (remove student, clear data, reset logo).

Auto-save on every change (debounced 120 ms) and on beforeunload / tab hide.

9. Known Constraints
No authentication — anyone with access to the browser can see/edit data.

Single-browser only — data does not sync across devices; use Backup/Restore to move.

localStorage size — typically 5–10 MB. Logo max ~900 KB recommended.

Excel import relies on the CDN library; offline use requires saving as CSV.

PDF export relies on jsPDF CDN; offline use is not supported for PDF.

No subject weighting — every core subject counts equally toward the grand total.

10. Recommended Usage Flow
Settings → set PA and Term Theory Max, Internal Max, Pass Mark, and grade bands.

Settings → optionally upload the school logo.

Students → add students manually or bulk import, per grade.

Marks Entry → select grade + exam → enter theory and internal marks, tick AB for absentees.

Results → verify per-subject grades, grand total, overall grade, rank.

Results → apply filters and column toggles as needed.

Results → click Excel or PDF to export.

Settings → click Backup regularly to save a JSON snapshot.

11. File Summary
File	Role
index.html	The entire application (HTML + CSS + JS in one file)
sjik.jpg	Default school logo (optional; can be replaced via Settings upload)
Backup JSON	Exported data snapshot (students + marks + settings)
