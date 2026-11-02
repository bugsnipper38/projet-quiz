# TeachTech — Codex Implementation Specification

## Project goal
Build a small, fully working educational web application for teachers to manage students, classes, assessments, grades, and personalized feedback. This is a local-first MVP, not just a UI mockup.

## Technology stack
- Frontend: React + Vite
- Styling: Tailwind CSS
- Backend: Node.js + Express
- Database: SQLite
- ORM: Prisma
- Navigation: React Router
- Icons: Lucide React
- Authentication: secure server-side sessions with hashed passwords

## MVP scope
Only a **Teacher** role is implemented initially. Teachers can manage their own classes, students, assessments, grades, and feedback. Structure the code so student accounts can be added later, but do not implement student login in this version.

## Functional requirements

### 1. Authentication
- Login with email and password; logout.
- Validate form inputs and display helpful errors.
- Protect frontend routes and backend endpoints.
- Store password hashes, not plaintext passwords.
- Use secure session cookies (HttpOnly, SameSite; Secure in production), session expiration, and CSRF protection for cookie-authenticated mutations.

### 2. Dashboard
Show total students, total classes, total assessments, overall average, recent grading activity, recent assessments, and a grade-distribution chart. Scope all results to the logged-in teacher.

### 3. Students
- Searchable table with add, edit, delete, and profile views.
- Fields: ID, first name, last name, optional email, registration number.
- Profile includes enrollments, grade history, and feedback.
- Confirm destructive actions and handle referenced records safely.

### 4. Classes
- Create, edit, delete classes.
- Enroll existing students; remove enrollments.
- View class rosters.
- Class fields: ID, name, description, teacher ID.

### 5. Assessments
- Create, edit, delete assessments belonging to a class.
- Fields: ID, title, description, type (Quiz, Exam, Assignment, Project), date, maximum grade, class ID.
- Maximum grade must be a positive number; common values are 20 and 40.

### 6. Grades and feedback
- Select an assessment and show students enrolled in its class.
- Enter/update one score and personalized feedback per student.
- Score is optional until graded, but when present must be numeric between 0 and the assessment's maximum grade.
- Prevent duplicate records for the same student and assessment.
- Persist feedback and grade updates; display save success/errors.
- Do not permit grading a student who is not enrolled in the assessment's class.

### 7. Reports
- Student performance, class averages, grade distribution.
- Filter by class and assessment.
- Normalize each recorded score to /20 as `(score / maxGrade) * 20`; exclude ungraded scores from averages.
- Define overall average as the arithmetic mean of recorded normalized grades. Show “No grades yet” when none exist.

## Database entities and constraints

**Teacher**: id, name, email (unique), passwordHash, createdAt.

**Student**: id, firstName, lastName, email (nullable), registrationNumber (unique), createdAt.

**Class**: id, name, description, teacherId (FK to Teacher).

**Enrollment**: id, studentId (FK), classId (FK); unique(studentId, classId).

**Assessment**: id, title, description, type, date, maxGrade, classId (FK).

**Grade**: id, assessmentId (FK), studentId (FK), score (nullable), feedback (nullable), updatedAt; unique(assessmentId, studentId).

Use appropriate indexes and carefully selected referential actions. Every class, assessment, enrollment, and grade API operation must verify teacher ownership through the related class. Student access must also be restricted to students in the teacher's classes (or students the teacher is authorized to manage). Avoid exposing unrelated student data.

## UI/UX requirements
- Modern, clean educational SaaS appearance with white/light-gray surfaces and blue/violet accents.
- Responsive sidebar, rounded cards, readable typography, mobile-friendly tables/forms.
- Loading, empty, success, and error states.
- Accessible labels, keyboard navigation, confirmation dialogs, and toast notifications.
- Primary pages: Login, Dashboard, Students, Student Profile, Classes, Class Details, Assessments, Grade Entry, Reports.

## Suggested project structure
```text
teachtech/
  frontend/
    src/
      components/
      pages/
      hooks/
      services/
      App.jsx
  backend/
    prisma/
      schema.prisma
      seed.js
    src/
      routes/
      controllers/
      middleware/
      services/
      app.js
      server.js
    tests/
  README.md
```
Adapt this structure if the repository already has established conventions.

## Step-by-step implementation phases

### Phase 1 — Setup
1. Inspect the repository before changing files.
2. Create a concise implementation checklist.
3. Initialize React/Vite frontend and Express backend if absent.
4. Configure Tailwind, routing, linting, environment variables, and Prisma/SQLite.
5. Create migrations and a repeatable seed script.

**Verification:** dependencies install, database migrates, and frontend/backend start locally.

### Phase 2 — Authentication and authorization
1. Implement Teacher model and hashed-password authentication.
2. Add session creation, logout, protected routes, and CSRF protection.
3. Implement teacher ownership checks on all protected APIs.

**Verification:** valid login works; invalid credentials and unauthorized requests are rejected.

### Phase 3 — Layout and student/class management
1. Build navigation, dashboard shell, and responsive layout.
2. Implement student CRUD and search.
3. Implement class CRUD, roster management, and enrollment.
4. Add empty states and destructive-action confirmations.

**Verification:** CRUD works and data persists after refresh.

### Phase 4 — Assessments, grades, feedback
1. Implement assessment CRUD.
2. Build grading table with score and feedback inputs.
3. Add server-side grade validation, enrollment checks, and upsert behavior.
4. Show save status and recoverable error messages.

**Verification:** grades/feedback persist; invalid scores and duplicate records are prevented.

### Phase 5 — Dashboard and reports
1. Calculate normalized /20 scores and averages.
2. Build dashboard statistics and chart.
3. Build filtered student/class reports.

**Verification:** calculations are correct for mixed /20 and /40 assessments and missing grades.

### Phase 6 — QA and handoff
1. Run all available tests, lint/build commands, and fix failures.
2. Seed demo data and verify the full user journey.
3. Document installation, environment variables, database migration, seeding, testing, and startup commands in README.
4. Summarize what was implemented and any remaining limitations.

## Required test scenarios
- Successful and failed login; logout; expired/invalid session.
- Unauthorized access and attempts to access another teacher's data.
- Student create/edit/delete and class enrollment.
- Assessment create/edit.
- Valid, negative, nonnumeric, and above-maximum scores.
- Ungraded (null) score behavior.
- Grade update does not create duplicate records.
- Feedback persists after refresh and new login.
- Grade rejected if student is not enrolled in the assessment's class.
- Correct normalized averages across assessments with different maximum scores.
- Empty-state and error-state behavior.

## Demo data
Seed one demo teacher, three classes, fifteen fictional students, several assessments, and sample grades/feedback. Use fictional student information only. Document demo credentials in the **local development README** and clearly state they must be changed or disabled outside development.

## Definition of done
A teacher can log in, create a class, add/enroll students, create an assessment, assign grades, write feedback, log out, log back in, and see all saved information unchanged. The app starts locally, database migrations work, core tests pass, and the README contains reproducible instructions.

## Instructions to Codex
- Implement the actual working application, not a static prototype.
- Follow phases in order; verify each before proceeding.
- Prefer simple, maintainable code and sensible defaults over unnecessary dependencies.
- Do not use real student data.
- Do not claim commands or tests succeeded unless you actually ran them.
- If an operation cannot be executed, explain why and provide the exact command for the user.
- At the end, report files created, commands run, tests passed/failed, and how to launch the application.

## Future enhancements (not part of MVP)
Student login and private grade viewing, CSV import/export, printable reports, notifications, teacher-defined grading rubrics, and deployment.
