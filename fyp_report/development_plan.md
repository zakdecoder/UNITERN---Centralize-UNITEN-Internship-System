# UNITERN – Development Plan

**Project:** UNITERN: Centralized UNITEN Internship System
**Basis:** `requirement.md` (FR01–FR15, NFR1–NFR9, BR01–BR12)
**Stack:** Laravel (PHP) · React + Inertia + Tailwind CSS · MySQL

> **Assumption:** the report does not give the FYP2 calendar. The plan uses a 14-week build counted from the first week of FYP2 (W1–W14). Adjust the week numbers to the official UNITEN schedule.

---

## 1. Approach

- **Iterative delivery in vertical slices.** Each sprint (2 weeks) ends with a working, demoable feature set that is deployed to a staging server and shown to the supervisor.
- **Order follows dependency and risk.** Authentication, roles and sessions come first because every other module depends on them. The submission and monitoring core comes next because it solves the highest-value problems (P3, P4). Admin tooling and polish come last.
- **Server-side authorization from day one** (Laravel policies and middleware), covered by automated tests, because the role rules are the most error-prone part of the specification.
- **Configurable values** (deadlines, file types, size limit, VL limit, reminder days) live in a `system_settings` table, not in code.

## 2. Architecture and Conventions

| Area | Decision |
|------|----------|
| Layers | Presentation (Inertia pages per role) → Application (Laravel services/controllers, middleware) → Data (MySQL + file storage) |
| Auth | Laravel Breeze (React + Inertia starter) as base; email + password; force password change on first login |
| Authorization | Role middleware plus Policies; permission table seeded with defaults (editable by Admin per FR15) |
| Files | Laravel `Storage` on a private disk; download through authorized controller routes only; never public URLs |
| Jobs / schedule | Laravel Scheduler: reminder emails, overdue marking, daily backup |
| Notifications | Laravel notifications: database channel (in-system) and mail channel (SMTP) |
| Logging | Activity log written by a single service or model observer; append-only |
| Code layout | Feature folders: `Account`, `Session`, `Announcement`, `Repository`, `Submission`, `Assignment`, `Monitoring`, `Admin` |
| Testing | Pest/PHPUnit feature tests for every route and role; Laravel Dusk or Playwright for key flows |
| Quality | Pint (PHP style), ESLint + Prettier, GitHub Actions running tests on each push |
| Environments | Local (Sail/XAMPP) → staging (supervisor demo, UAT) |
| Version control | GitHub, `main` protected, short-lived feature branches, pull request per feature |

## 3. Data Model (first draft)

The report's ERD section is empty, so this draft is proposed for Chapter 4.

```
users(id, name, university_id, email, password, role, department_id, is_active, must_change_password)
departments(id, name)
internship_sessions(id, name, start_date, end_date, status[active|disabled|archived])
session_students(id, session_id, student_id, vl_id NULL, department_id)   -- unique(session_id, student_id)
announcements(id, author_id, session_id NULL, department_id NULL, target_roles, title, body, created_at)
notifications(...)                                                          -- Laravel standard table
repository_folders(id, owner_id, session_id NULL, name, is_enabled, vl_scope_id NULL)
folder_role_access(folder_id, role)
repository_files(id, folder_id, name, path, size, mime, is_enabled, uploaded_by)
submission_windows(id, session_id, department_id NULL, title, opens_at, deadline, is_open)
submissions(id, window_id, student_id, file_path, original_name, size, submitted_at)   -- one active per window+student
complaints(id, student_id, session_id, subject, body, created_at)
activity_logs(id, actor_id, action, subject_type, subject_id, meta JSON, created_at)
system_settings(key, value)                                                -- file types, size, default deadline, VL limit, reminder days, email on/off, maintenance
role_permissions(role, permission, allowed)
```

Key constraints:
- `session_students` unique on (session, student) → one VL per student per session.
- VL limit enforced inside a database transaction with row locking (FR13 AC2).
- Status (Not Submitted / Submitted / Overdue) is **derived** from window deadline + presence of a submission, not stored.
- Completion % = submitted windows ÷ windows applicable to that student (decision needed, see `requirement.md` §10.1).

## 4. Work Breakdown and Schedule

### Phase 0 – Close FYP1 gaps (before W1)
- Resolve the open decisions in `requirement.md` §10 with the supervisor and coordinator (especially window scoping and role overlap).
- Complete Chapter 4: ERD, use case diagram, activity/sequence diagrams for submit, VL selection, and announcement; page wireframes per role.
- Fix the report housekeeping items (abstract, acknowledgement, references, tables/figures lists, objective numbering).
- Set up repository, CI, staging host.

**Exit:** signed-off requirements, ERD, wireframes, working "hello world" deployment.

### Sprint 1 (W1–W2) – Foundation, accounts, roles
| Task | FR / NFR |
|------|----------|
| Laravel + Inertia + React + Tailwind scaffold, layouts per role | NFR1, NFR7 |
| Migrations and seeders (departments, default settings, Admin account) | – |
| Login/logout, change password, forced change on default password | FR01, NFR2 |
| Role middleware, policies, permission seeding | FR01, NFR2 |
| Staff account management (Admin), role assignment rules, activate/deactivate, force logout, reset password | FR02 |
| Activity log service (used by every later feature) | NFR8 |
| Feature tests for role matrix | NFR2 |

**Demo:** log in as each role and see the correct empty dashboard; rule tests for BR05 pass.

### Sprint 2 (W3–W4) – Sessions, departments, students
| Task | FR |
|------|----|
| Session CRUD, enable/disable, archive (view-only) | FR04 |
| Department CRUD (HOIT/Admin only) | FR04 |
| Add student to session (single form) | FR03 |
| CSV import with validation report, dry-run preview | FR03 |
| "Not registered" screen for students outside an active session | FR01, BR03 |
| Maintenance mode | FR15 |

**Demo:** import a sample CSV of 100 students; PC cannot delete a department.

### Sprint 3 (W5–W6) – Announcements, notifications, repository
| Task | FR |
|------|----|
| Announcement create/delete with role/department/session targeting | FR05 |
| Announcement list and detail for Student, VL, PC | FR06 |
| In-system notification center, unread badge | FR06 |
| Repository folders, role access, upload, enable/disable | FR07 |
| Authorized download endpoint | FR08, NFR2 |
| VL-scoped folders | FR07 |

**Demo:** HOIT posts announcement and uploads template → PC and students see it; VL cannot access another VL's folder.

### Sprint 4 (W7–W8) – Submission core (highest value)
| Task | FR |
|------|----|
| Submission window CRUD, open/close, deadline | FR09 |
| Student upload with type/size validation, confirmation, due date and limit shown | FR10, NFR1 |
| Delete and resubmit before deadline | FR10 |
| Derived status display (Not Submitted / Submitted / Overdue) | FR11 |
| Submission notification to student and VL | FR06 |
| Failed-upload error handling, no partial records | NFR5 |

**Demo:** full student submit → delete → resubmit → deadline-pass flow.

### Sprint 5 (W9–W10) – VL assignment and monitoring
| Task | FR |
|------|----|
| VL search/select unassigned students, limit 10, concurrency-safe | FR13 |
| Reassign by Admin/HOIT/PC, change limit globally or per VL | FR13 |
| VL view: assigned students, previous students, submission progress, download files | FR13, FR14 |
| HOIT/PC monitoring: per-student status, completion %, per-VL and per-department views, overdue filter | FR14 |
| PC notification on VL selection | FR06, BR06 |

**Demo:** two VLs race for the same student; coordinator dashboard lists overdue students.

### Sprint 6 (W11–W12) – Complaints, scheduler, admin tools
| Task | FR |
|------|----|
| Complaint form and inbox for PC/HOIT | FR12 |
| Scheduler: reminder emails before deadline, overdue marking | FR06, FR15 |
| Reminder settings, system settings (file type, size, default deadline, VL limit) | FR15, NFR7 |
| Role permission editor (fixed vs editable rules) | FR15 |
| Activity log viewer with filters and export | FR15, NFR8 |
| Backup/restore, storage usage, archived session management | FR15, NFR5, NFR9 |

**Demo:** reminder email arrives; backup then restore on staging.

### Sprint 7 (W13–W14) – Hardening, testing, UAT
| Task | Output |
|------|--------|
| Cross-browser and mobile pass | NFR6 |
| Performance checks with seeded data (e.g., 500 students, 5,000 files) | NFR3 |
| Security review (authorization, upload handling, rate limiting, HTTPS) | NFR2 |
| Bug fixing from system test | – |
| UAT with students, VLs, coordinator; SUS/UAT questionnaire | Objective 4 |
| Final report chapters 5–6, user manual, deployment notes | Deliverables |

## 5. Testing Strategy

| Level | What | Tooling |
|-------|------|---------|
| Unit | Status derivation, completion %, VL limit logic, CSV parsing | Pest/PHPUnit |
| Feature (HTTP) | Every route × every role (allowed/denied) based on the role matrix | Pest |
| Concurrency | Simultaneous VL selection of the same student | Pest with parallel requests or DB transaction test |
| End-to-end | Login → submit file; HOIT announcement → student sees it; VL selection | Playwright/Dusk |
| Non-functional | Load time and upload time (NFR3), backup/restore (NFR5), browser matrix (NFR6) | Lighthouse, manual |
| UAT | Scenario tasks per role with success/failure and satisfaction scores | Google Forms, observation sheet |

**UAT design (Objective 4):**
- Participants: 10–15 students, 3–5 VLs, HOIT/PC (target ≥ 20 total, matching the report's survey-based approach).
- Task sets: A (announcements and repository), B (submission and status), C (VL assignment), D (monitoring).
- Metrics: task completion rate, time on task, SUS score, free-text feedback.
- Success target (proposed): ≥ 90% task completion and SUS ≥ 68.

## 6. Definition of Done (per feature)
1. Acceptance criteria in `requirement.md` pass.
2. Feature tests for allowed and denied roles are green.
3. Actions that need auditing write to the activity log.
4. Responsive layout checked at mobile and desktop widths.
5. Merged through pull request with CI passing and deployed to staging.

## 7. Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| Requirements are still ambiguous (window scoping, role overlap) | Rework in Sprint 4–5 | Resolve §10 of `requirement.md` in Phase 0 |
| Scope too large for one student (15 FRs, 5 roles) | Slippage | Priorities: Sprints 1–5 are the must-have core; FR15 items (permission editor, backup/restore UI) are the first to descope |
| Role/permission logic bugs | Security failures | Policy-based checks plus automated role matrix tests from Sprint 1 |
| Large file uploads fail or time out (20 MB within 30 s) | Poor UX | Tune PHP/nginx limits early; test in Sprint 4; chunk uploads only if needed |
| No access to real UNITEN infrastructure | Cannot host in production | Use a cloud/VPS staging server; document deployment for later hand-over |
| Backup/restore is hard to expose in a web UI | Delays | Implement via artisan command + simple UI trigger; restrict to Admin |
| Low UAT participation | Weak evaluation | Recruit in W10; use survey respondents who already agreed to help; schedule UAT slots early |
| Email deliverability for reminders | Reminders not received | Use a transactional SMTP service; test with real university mailboxes |
| Personal data and complaints stored | Privacy exposure | Minimal fields, access restricted to PC/HOIT/Admin, no public file URLs, retention policy documented |
| Illness or schedule clash | Missed milestones | Two-week buffer inside Sprint 7; weekly supervisor check-in |

## 8. Milestones and Deliverables

| Milestone | Week | Deliverable |
|-----------|------|-------------|
| M0 Design sign-off | Before W1 | Final requirements, ERD, wireframes, CI/CD |
| M1 Access control working | W2 | Auth, roles, activity log |
| M2 Sessions and students | W4 | Session, department, CSV import |
| M3 Information channel | W6 | Announcements, notifications, repository |
| M4 Submission core | W8 | Windows, submission, statuses |
| M5 Monitoring and assignment | W10 | VL assignment, coordinator dashboard |
| M6 Feature complete | W12 | Complaints, scheduler, admin tools |
| M7 Evaluation complete | W14 | System testing, UAT results, final report, demo |

## 9. Deployment Plan
- **Staging** from Sprint 1; auto-deploy `main` after tests pass.
- **Production-like config for UAT:** HTTPS, queue worker, scheduler cron (`schedule:run` every minute), daily DB + file backup, `.env` secrets outside the repository.
- **Seed data for demos:** one session, 3 departments, 1 Admin, 1 HOIT, 2 PC, 5 VL, 100 students, sample templates.
- **Handover pack:** README, environment setup, user manual per role, backup/restore guide.

## 10. Requirement Coverage Check

| Requirement | Sprint |
|-------------|--------|
| FR01 | 1 |
| FR02 | 1 |
| FR03 | 2 |
| FR04 | 2 |
| FR05, FR06 | 3 (notifications extended in 4–6) |
| FR07, FR08 | 3 |
| FR09, FR10, FR11 | 4 |
| FR12 | 6 |
| FR13, FR14 | 5 |
| FR15 | 2 (maintenance mode), 6 (rest) |
| NFR1–NFR9 | Built throughout; verified in Sprint 7 |
