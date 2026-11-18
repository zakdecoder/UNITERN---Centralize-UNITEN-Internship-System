# UNITERN – Requirements Specification

**Project:** UNITERN: Centralized UNITEN Internship System
**Author:** Muhammad Zakariyya bin Ahmad Radzif (BSW01085129)
**Supervisor:** Ts. Dr. Suhaimi bin Ab. Rahman
**Source:** FYP1 report v15 (Chapters 1–4)
**Stack (fixed by scope):** React + Inertia + Tailwind CSS · Laravel (PHP) · MySQL

---

## 1. Purpose and Problem

UNITEN undergraduates must complete a mandatory internship. Today the procedure is spread across the CCI Wix website (static files), SharePoint links, the ITS section of Student Info UNITEN, email, Telegram and WhatsApp. Logbooks and reports are sent to each supervisor by whatever channel they prefer.

UNITERN is a single web system that **manages** the internship procedure instead of just publishing documents about it.

| # | Problem | What UNITERN must do about it |
|---|---------|-------------------------------|
| P1 | Information and templates are scattered across several sources | One stage-organised file repository |
| P2 | Announcements go out through Telegram and email, with no searchable record | One announcement module with notifications |
| P3 | No single submission channel and no confirmation of receipt | Online submission with confirmation and status |
| P4 | Coordinators cannot see who is behind | Monitoring with completion percentage and overdue status |

Survey evidence (30 students who finished internship): information is found mainly via CCI Wix, email and WhatsApp; about 60% nearly missed a deadline, mostly because of no reminders or misinformation.

## 2. Objectives (from Chapter 1)

1. Review the current UNITEN internship procedure and existing systems (done: Chapter 2).
2. Build an announcement module so the coordinator publishes information and files in one place.
3. Build coordinator tooling to manage an internship per session: monitor submissions, and monitor student assignment to visiting lecturers.
4. Evaluate the system through system testing and User Acceptance Testing (UAT) with students, supervisors and the coordinator.

## 3. Scope

### 3.1 In scope
- Five roles: Admin, HOIT, PC, VL, Student
- Account provisioning by staff (manual and bulk CSV)
- Sessions, departments, archive
- Announcements and notifications (in-system, plus email reminders)
- File repository (folders, role-based access)
- Submission windows, student submission, status tracking
- VL ↔ student assignment (max 10 per VL)
- Monitoring and completion percentage
- Student complaints
- Admin: settings, permissions, activity log, backup/restore, maintenance mode

### 3.2 Out of scope
- Native mobile application (responsive web only)
- Integration with UNITEN login or the ITS portal / student data
- Automated document checking or verification
- AI features
- Integrated e-logbook (the CCI template is used manually)
- Company placement application and employer-facing features

### 3.3 Assumptions
- Accounts are created by staff or bulk-imported; student ID, email and department are collected outside the system.
- CCI templates (logbook, report, evaluation forms) and the internship timeline stay unchanged during development; deadlines are configurable values.

## 4. Users and Roles

| Role | Summary |
|------|---------|
| **Admin** | System owner. Creates staff accounts, assigns roles, controls settings, logs, backup, archived sessions. Can do everything HOIT can do. |
| **HOIT** (Head of Industrial Training) | Creates sessions and departments, adds students, assigns PC and VL, sends information to PC through announcements and repository. |
| **PC** (Program Coordinator) | Same day-to-day control as HOIT, except: may assign VL only, cannot create or delete departments, cannot create staff accounts. |
| **VL** (Visiting Lecturer) | Works with own students only: selects up to 10, tracks submissions, downloads files. |
| **Student** | Views announcements, downloads from repository, submits files, checks status, sends complaints. |

## 5. Business Rules

| ID | Rule |
|----|------|
| BR01 | Admin creates HOIT, PC and VL accounts before a session starts. Login is university email plus a default password. Staff accounts are reusable across sessions. |
| BR02 | Admin, HOIT or PC creates a student account when adding the student to a session (one by one or CSV). If the student already exists, they are only added to the new session. |
| BR03 | A student can use the system only after the account exists **and** they are in an active session; otherwise they see a "not registered" message. |
| BR04 | Every staff member and student has a department. Department is a **filter only** and never restricts access. |
| BR05 | Role assignment: Admin → HOIT, PC, VL. HOIT → PC, VL. PC → VL only. |
| BR06 | A VL selects students of the current session who have no VL yet, up to 10 (limit configurable by Admin, HOIT, PC). One student has exactly one VL. No approval step, but every selection is logged and PC is notified. |
| BR07 | Information flows HOIT → PC (announcements and repository), then PC → VL and students. |
| BR08 | HOIT and PC open and close submission windows and set deadlines. A student's completion percentage counts only windows that apply to that student. |
| BR09 | A student may delete a submitted file and resubmit only before the deadline and while the window is open. |
| BR10 | Submission status values are exactly: **Not Submitted, Submitted, Overdue**. |
| BR11 | Finished sessions are archived (view-only). Only Admin can delete an archived session. |
| BR12 | In maintenance mode only Admin can log in. |

## 6. Functional Requirements

Priorities are as proposed in the report (Table 3.3). Acceptance criteria are added here to make each requirement testable.

### FR01 – Authentication and role-based access (High) · All
Log in/out with university email and password; users see only their role's functions; students log in only if in an active session; users change their own password.
- **AC1** Wrong credentials are rejected with a generic error.
- **AC2** A student not in an active session sees the "not registered" message.
- **AC3** Every route is checked by role on the server (not only hidden in the UI).
- **AC4** Deactivated users cannot log in.

### FR02 – Manage staff account and role (High) · Admin, HOIT, PC
Admin adds/deletes HOIT, PC, VL by department. Admin, HOIT, PC assign roles and activate/deactivate access within BR05. Admin can force logout and reset a password to the default.
- **AC1** HOIT and PC cannot create or delete staff accounts.
- **AC2** PC attempting to assign PC or HOIT is refused.
- **AC3** Force logout invalidates the user's active session immediately.

### FR03 – Manage student account (High) · Admin, HOIT, PC
Create student accounts and add them to a session, individually or by CSV; remove a student from a session.
- **AC1** CSV import reports per-row success/failure (invalid email, duplicate ID, missing department).
- **AC2** An existing student is attached to the new session, not duplicated.
- **AC3** Removing a student from a session does not delete the account or historical data.

### FR04 – Manage session and department (High) · HOIT, PC, Admin
Create, delete, enable/disable sessions; create/delete departments (HOIT and Admin only); finished sessions are archived and view-only.
- **AC1** PC can manage sessions but cannot create or delete departments.
- **AC2** Archived session data cannot be edited by any role.

### FR05 – Manage announcements (High) · HOIT, PC
Post and delete announcements targeted by role, department and session.
- **AC1** Only users in the targeted role/department/session see the announcement.
- **AC2** Posting triggers notifications to the target audience.

### FR06 – View announcements and receive notifications (High) · Student, VL, PC
Announcements are listed in one place; notifications for new announcement, deadline and submission update (student: new announcement, deadline reminder, submission confirmation; VL: new submission, deadline reached, new announcement; PC: VL student selection and complaints).
- **AC1** Unread notification count is visible after login.
- **AC2** Email reminders are sent N days before deadline when enabled in reminder settings.

### FR07 – Manage file repository folder (High) · HOIT, PC, VL
Create/delete folders, choose accessible roles, upload files, enable/disable a folder or file. VL manages folders only for own students.
- **AC1** Disabled folders/files are invisible and non-downloadable to students and VLs.
- **AC2** A VL cannot see or modify another VL's folder.
- **AC3** Uploads obey the allowed file types and size limit from system settings.

### FR08 – Download from repository (High) · Student, VL
View and download documents in folders shared to the user's role (students also see their VL group's folders).

### FR09 – Manage submission windows (High) · HOIT, PC
Open/close a submission window with a deadline.
- **AC1** Closing a window blocks new uploads immediately.
- **AC2** The default deadline comes from system settings and can be overridden per window.

### FR10 – Submit and delete file (High) · Student
Submit while the window is open and receive a confirmation; delete and resubmit before the deadline.
- **AC1** The due date and file size limit are displayed beside the upload field (NFR1).
- **AC2** Invalid type/size produces a clear error and no partial record.
- **AC3** A confirmation (on screen and as notification) is issued on success.
- **AC4** Delete requires a confirmation prompt.

### FR11 – Display submission status (High) · Student
Show Not Submitted / Submitted / Overdue for each required submission.
- **AC1** Status is Overdue automatically once the deadline passes with no submission.

### FR12 – Send complaint (High) · Student, PC, HOIT
Student sends a named complaint (e.g. workplace bullying) to PC; HOIT also receives it.
- **AC1** The complaint is visible only to the sender, PC and HOIT (and Admin).
- **AC2** PC and HOIT are notified.

### FR13 – Assign student to VL (High) · VL, Admin, HOIT, PC
VL searches and selects unassigned students of the current session (optional department filter), up to the limit, without approval and can remove from own list. Admin/HOIT/PC assign, reassign, view, and change the limit (all VL or one VL).
- **AC1** The 11th selection is refused with a message.
- **AC2** Two VLs selecting the same student concurrently: exactly one succeeds.
- **AC3** Each selection is written to the activity log and PC is notified.
- **AC4** VL can view own previous-session students (view only).

### FR14 – Monitor submission and completion (High) · VL, HOIT, PC
VL views assigned students' submission status and deadlines and downloads submitted files. HOIT and PC view submission status and completion percentage per student and across VL and departments.
- **AC1** Completion % = submitted windows ÷ applicable windows for that student.
- **AC2** Overdue students can be filtered/listed.

### FR15 – System administration (Medium) · Admin (HOIT for activity log)
Maintenance mode, system settings (file types, file size, default deadline, default VL limit), role permissions, reminder settings, activity log with filter and export, backup/restore with storage usage, archived session management.
- **AC1** Activity log can be filtered by role, department or student and exported.
- **AC2** Backup and restore can be run from the UI.

### 6.1 Role × feature matrix (derived from Tables 1.1–1.5)

| Capability | Admin | HOIT | PC | VL | Student |
|---|:-:|:-:|:-:|:-:|:-:|
| Create/delete staff accounts | ✓ | | | | |
| Assign roles | HOIT, PC, VL | PC, VL | VL | | |
| Activate/deactivate access | ✓ | ✓ (staff, student) | ✓ (student, VL) | | |
| Force logout / reset password | ✓ | | | | |
| Create/delete session | ✓ | ✓ | ✓ | | |
| Create/delete department | ✓ | ✓ | | | |
| Add/remove student to session | ✓ | ✓ | ✓ | | |
| Change VL student limit | ✓ | ✓ | ✓ | | |
| Post/delete announcement | ✓ | ✓ | ✓ | | |
| View announcement / notifications | | | ✓ | ✓ | ✓ |
| Manage repository folders | ✓ | ✓ | ✓ | own students | |
| Download from repository | | | | ✓ | ✓ |
| Open/close submission window | ✓ | ✓ | ✓ | | |
| Submit / delete file | | | | | ✓ |
| Download submitted files | | | | ✓ (assigned) | |
| Assign / reassign VL students | ✓ | ✓ | ✓ | self-select | |
| Monitor progress | ✓ | ✓ | ✓ | assigned | own status |
| Receive complaints | ✓ | ✓ | ✓ | | sends |
| Activity log | ✓ (all) | ✓ (below HOIT) | own monitor view | | |
| Settings, backup, maintenance, delete archive | ✓ | | | | |
| View archived session | ✓ | ✓ | ✓ | own previous | |

## 7. Non-Functional Requirements

| ID | Attribute | Requirement | How to verify |
|----|-----------|-------------|---------------|
| NFR1 | Usability | Consistent layout across roles; due date and size limit beside every upload field; confirmation before delete | UAT checklist, heuristic review |
| NFR2 | Security | Passwords hashed; HTTPS; server-side role checks | Authorization tests per route, config review |
| NFR3 | Performance | Page load ≤ 4 s; 20 MB upload ≤ 30 s | Timed tests with seeded data |
| NFR4 | Availability | ≥ 99% uptime during submission periods | Uptime monitor during UAT |
| NFR5 | Reliability | No loss of submitted files; daily DB backup; clear upload error messages | Backup/restore drill, failed-upload test |
| NFR6 | Compatibility | Current Chrome, Edge, Firefox, Safari; usable on mobile browsers | Cross-browser and responsive test |
| NFR7 | Maintainability | Standard Laravel/React structure; deadlines, file types, size limits configurable | Code review, settings change without redeploy |
| NFR8 | Auditability | Account creation, role assignment, VL selection (and other key actions) logged | Log inspection tests |
| NFR9 | Data retention | Archived data kept; only Admin deletes archived sessions | Permission tests |

## 8. Architecture Constraints (from Figure 4.x)

Four layers: Users (browser, HTTPS) → Presentation (React + Inertia + Tailwind; Student, VL, HOIT/PC, Admin pages) → Application (Laravel; middleware for login and role check on every request; modules: Account and Role, Session and Department, Announcement and Notification, File Repository, File Submission, VL Assignment, Monitoring and Completion, Activity Log, System Settings, Scheduler for reminder/overdue with SMTP email) → Data (MySQL, file storage, backup).

## 9. Suggested Data Entities

The ERD section of the report is still empty. This is a starting point derived from the requirements:

`users`, `roles` / `permissions`, `departments`, `internship_sessions`, `session_students` (student ↔ session, VL assignment, department), `announcements` (+ target role/department/session), `notifications`, `repository_folders` (+ role access, owner), `repository_files`, `submission_windows` (session, title, deadline, open/closed, applicable department), `submissions` (window, student, file, submitted_at), `complaints`, `activity_logs`, `system_settings`.

## 10. Gaps, Ambiguities and Decisions Needed

These are issues found in the proposal. Each affects design or testing, so they are best settled before or early in FYP2.

1. **"Windows that apply to that student" is undefined (BR08).** Decide how a window is scoped: whole session, by department, or by student group. This drives completion % and Overdue.
2. **Submission goes "to VL", but windows are set by HOIT/PC.** Confirm whether a submission is visible to PC/HOIT as well as the VL (FR14 says yes) and whether files are stored per window or per VL.
3. **No review/acknowledge step.** Chapter 2 discusses statuses like "pending review/approved", but BR10 limits status to three values. State explicitly that VL does not approve submissions.
4. **Objective numbering is inconsistent.** Chapter 1 has four objectives with UAT as Objective 4; Chapter 3 refers to UAT as "Objective 3"; the Gap Analysis maps four gaps to objectives in a different order. The Gap Analysis also says "admin" where HOIT/PC are meant.
5. **Role overlap.** PC can create/delete sessions (Table 1.3) while the scope rules say HOIT creates sessions. Confirm.
6. **Role permissions editable by Admin** (FR15) can conflict with BR05 and the matrix above. Define which permissions are editable and which are fixed.
7. **Email reminders:** the survey asks how students want notifications; results are not yet reported. Confirm channels (in-system + email only, no Telegram).
8. **Complaint handling:** the report says "named" complaint. No status or response flow is specified; assume view-only receipt unless extended.
9. **File security:** virus scanning is not in scope; decide whether allowed types plus size limits are sufficient.
10. **Default passwords:** specify forced change on first login (NFR2 implies it but does not say).
11. **Data protection:** the system stores student IDs, emails and complaints. Mention PDPA-style handling and retention in the report.
12. **FR12 wording** ("bully in internship") should become a neutral category such as "workplace complaint".
13. **Report housekeeping (not a requirement):** abstract and acknowledgement are still template text; the ERD section is empty; the references include placeholders (Quintua initials/title, Wan Abdul Rahman "other authors"); the List of Tables and Figures and abbreviations are template leftovers; the Results section does not yet include figures and charts that it describes; the Table of Contents lacks Chapters 3 and 4 detail.

## 11. Traceability (Objective ↔ Requirement ↔ Test)

| Objective | Requirements | Verified by |
|-----------|--------------|-------------|
| Obj 2 – Announcement module | FR05, FR06, FR07, FR08 | System tests ST-ANN, UAT task set A |
| Obj 3 – Session, submission and assignment management | FR02–FR04, FR09–FR14 | System tests ST-SUB, ST-ASG, UAT task sets B–D |
| Obj 4 – Evaluation | FR01–FR15, NFR1–NFR9 | Full test plan and UAT |
