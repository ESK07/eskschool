# ESK School

A full-stack, multi-school management platform inspired by YoSchool: student records, academics, attendance, timetables, report cards, fees and invoicing, payroll, messaging and reporting — with a professional **Pro mode** for staff and parents and a playful **Kid mode** for younger learners.

| Layer     | Stack |
|-----------|-------|
| Database  | PostgreSQL 16 (`server/db/schema.sql`) |
| API       | Node 18+, Express, `pg`, JWT auth, role-based access, Zod validation |
| Frontend  | React 18, Vite, Tailwind CSS, Framer Motion, TanStack Query, Recharts, lucide-react |

## Quick start (local)

Prerequisites: Node 18+ and Docker (for Postgres).

```bash
# 1. Start PostgreSQL (port 5433)
docker compose up -d db

# 2. API — installs deps, applies schema, seeds two demo schools, starts on :4000
cd server && npm install && npm run migrate && npm run seed && npm run dev

# 3. Frontend — in a second terminal, starts on :5173 and proxies /api to :4000
cd client && npm install && npm run dev
```

Open http://localhost:5173 and sign in with a demo account.

### Everything in Docker

```bash
docker compose --profile full up --build
```

Runs Postgres, the API (auto-migrates + seeds) and the built frontend behind nginx on http://localhost:5173.

## Demo accounts

Password for every demo account is `password`.

| Email                    | Role       | Notes |
|--------------------------|------------|-------|
| admin@esk.school         | Admin      | Member of **both** schools — use the school switcher in the top bar |
| teacher@esk.school       | Teacher    | ESK Primary; sees "My lessons" and their classes |
| accountant@esk.school    | Accountant | Finance, invoices, payments, payroll |
| student@esk.school       | Student    | ESK Primary — opens in **Kid mode** |
| parent@esk.school        | Parent     | Linked to the demo student |
| teacher.high@esk.school  | Teacher    | ESK High School |
| student.high@esk.school  | Student    | ESK High School (Pro mode) |

You can also register a brand-new school at `/register`; it comes pre-configured with an academic year, three terms and a grading scale.

## Features

- **Multi-school** tenancy with a school switcher and a term switcher that re-scopes every page.
- **People**: students (admissions, profiles, guardians, badges), guardians, staff (HR records, salaries).
- **Academics**: classes & subject allocation, subjects, daily attendance register with analytics, assessments with a keyboard-friendly gradebook, weighted report cards with class positions, homework with submissions, lesson coverage, weekly timetable with clash detection.
- **Finance**: fee structures per term/class, one-click invoice generation, invoices with line items, payments (cash, bank, mobile money…), finance dashboard, payroll runs with payslips.
- **Communication**: announcements by audience, events calendar, private messaging with broadcasts.
- **Reporting**: academic, enrolment, attendance and finance reports with CSV export; audit log.
- **Kid mode**: bigger type and touch targets, a rounded playful theme, mascot avatars, points, badges, class leaderboard and confetti when homework is marked done. Adults can toggle it from the top bar too.
- **Fluid UI**: page transitions, staggered lists, spring modals, animated counters and progress rings, light/dark/system theme, per-school brand colour.

## Project layout

```
esk-school/
├── docker-compose.yml
├── server/
│   ├── db/schema.sql          # PostgreSQL schema
│   └── src/
│       ├── index.js           # Express app
│       ├── migrate.js         # applies schema (--reset drops everything)
│       ├── seed.js            # demo data (--force to add to an existing DB)
│       ├── middleware/auth.js # JWT, school scoping, role guard
│       ├── lib/crud.js        # school-scoped CRUD router factory
│       └── routes/            # auth, school, people, academics, attendance, finance, timetable, comms, hr, reports, dashboard
└── client/
    └── src/
        ├── lib/               # api client, auth + UI providers, hooks, utils
        ├── components/        # UI kit, app shell, generic CrudPage, confetti
        └── pages/             # role-aware pages (staff, kid, parent)
```

## API overview

All endpoints live under `/api`. Everything except `/api/auth/*` and `/api/health` requires `Authorization: Bearer <jwt>` **and** an `X-School-Id` header; the server verifies the user belongs to that school and applies role rules (admin, teacher, accountant, student, parent). Students and parents only ever see their own learners' data.

Key routes: `auth/*`, `school`, `terms`, `members`, `students`, `guardians`, `staff`, `classes`, `subjects`, `attendance`, `assessments/:id/scores`, `homework/:id/submissions`, `lesson-coverage`, `timetable`, `fee-structures`, `invoices` (+`/generate`), `payments`, `finance/summary`, `payroll`, `announcements`, `events`, `messages`, `achievements`, `leaderboard`, `reports/*`, `dashboard`, `audit`.

## Configuration

`server/.env` (see `.env.example`):

```
DATABASE_URL=postgres://esk:esk_secret@localhost:5433/esk_school
JWT_SECRET=change-me-in-production
PORT=4000
CLIENT_ORIGIN=http://localhost:5173
```

Reset and re-seed the database at any time with `npm run db:reset` in `server/`.
