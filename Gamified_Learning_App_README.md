# Gamified Learning App

A web platform that turns college coursework into structured, task-based learning — students complete tasks to earn XP, level up, unlock badges, maintain streaks, compare progress with their batch, and get help from mentors.

> 📋 **Status: Planning / Pre-Development** — the requirements and architecture are fully specified (see [SRS](#documentation)); implementation has not started yet. Deployment planned post-MVP.

---

## Product Vision

Students should always know what to learn next. This platform makes that visible and motivating: complete tasks → earn XP → level up → unlock badges → keep a streak → see where you stand in your batch → ask a mentor when stuck. Administrators run the entire program — tasks, badges, users, announcements — without touching code.

## Features (v1 Scope)

- 🔐 **Authentication & roles** — email/password signup with verification, JWT sessions, Student / Mentor / Administrator roles
- ✅ **Task-based learning** — admin-created tasks scoped by batch/branch, student checklists, idempotent completion, admin review/revocation with audit trail
- 🎮 **Gamification engine** — XP ledger, configurable level thresholds, badge rules, daily streaks with milestone notifications
- 🏆 **Batch-scoped leaderboard** — deterministic ranking (XP → earliest achievement → user ID), no private data exposed
- 🧑‍🏫 **Mentoring** — students request approved mentors; accepted pairs unlock real-time one-to-one chat (Socket.IO)
- 🔔 **Notifications & announcements** — in-app notifications for mentor decisions, level-ups, streaks; admin-published announcements
- 🛠️ **Admin console** — manage users, tasks, badges, mentor approval, announcements, and view operational metrics
- 📜 **Audit logging** — role changes, task edits, XP adjustments, and badge configuration are all logged with actor/action/target

**Out of scope for v1:** native mobile apps, payments/enrollment, grading integration, global cross-college leaderboard, video/group chat.

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React (SPA), React Router |
| Backend | Express (REST API), Socket.IO server |
| Database | PostgreSQL (migrations, transactions, constraints, indexes) |
| Auth | JWT, bcrypt/Argon2 password hashing |
| Real-time | Socket.IO (mentor-student chat) |
| Email | Transactional provider (verification, notifications) |
| Deployment (planned) | Static frontend host + API service host + managed PostgreSQL, HTTPS, centralized logging |

## Architecture

```
React SPA  ──HTTP──▶  Express REST API  ──▶  PostgreSQL
    │                        │
    └──Socket.IO────▶  Socket.IO Server (auth'd rooms, mentor-student pairs)

Task completion → XP ledger entry (transactional) → level recalculation → badge evaluation → notification
```

## Data Model (core tables)

| Table | Purpose |
|---|---|
| `users` | Identity, role, batch/branch, total XP, level |
| `tasks` | Admin-defined tasks with XP value and batch/branch scope |
| `user_tasks` | Per-student task status (unique per student-task pair) |
| `badges` / `user_badges` | Badge definitions and awards (unique per user-badge) |
| `xp_ledger` | Immutable XP award history — source of truth for total XP |
| `mentor_requests` | Student → mentor requests and their status |
| `messages` | Mentor-student chat history |
| `streaks` | Current and longest streak per user |
| `notifications` | In-app notifications |
| `announcements` | Admin-published announcements |

Full schema, relationships, and constraints are defined in the SRS (Section 6).

## API Outline

| Area | Example endpoints |
|---|---|
| Auth | `POST /auth/signup`, `POST /auth/login`, `POST /auth/verify-email`, `GET /auth/me` |
| Profile | `GET /users/me`, `PATCH /users/me` |
| Tasks | `GET /tasks`, `POST /tasks/:id/complete`, `CRUD /admin/tasks` |
| Gamification | `GET /progress/me`, `GET /leaderboard?batch=me`, `GET /badges` |
| Mentoring | `GET /mentors`, `POST /mentor-requests`, `PATCH /mentor-requests/:id` |
| Messages | `GET /conversations/:id/messages`; Socket events: `join_conversation`, `message:send`, `message:new` |
| Notifications | `GET /notifications`, `PATCH /notifications/:id/read` |
| Admin | `GET /admin/users`, `CRUD /admin/badges`, `CRUD /admin/announcements`, `GET /admin/metrics` |

## Key Business Rules

- Task completion, XP award, level recalculation, and badge evaluation run in a **single transaction** — duplicate XP cannot occur.
- Only *published* tasks matching a student's batch/branch scope can be completed.
- XP values and level thresholds are **admin-configurable**, not hard-coded.
- Mentor-student chat rooms exist **only** after a request is accepted, and close when the relationship is disabled.
- All timestamps stored in UTC; streak logic uses the college's configured time zone.

## Non-Functional Targets

- **Security:** HTTPS in production, hashed passwords, JWT expiry/rotation, server-side role checks, rate-limited auth endpoints, parameterized queries
- **Privacy:** minimum data collection, no emails on leaderboards, role-limited admin access
- **Performance:** dashboard/task/leaderboard endpoints under 500ms at pilot load
- **Reliability:** transactional XP awarding, daily DB backups, structured error logging
- **Compatibility:** latest two versions of Chrome, Edge, Firefox, Safari; responsive down to 360px

## Development Roadmap

This project follows a 12-phase build plan (full detail in the SRS):

- [ ] **Phase 0** — Confirm product rules (roles, XP curve, badge rules, batch structure)
- [ ] **Phase 1** — Project setup (repo, frontend/backend scaffolding, PostgreSQL, CI)
- [ ] **Phase 2** — Database schema and migrations
- [ ] **Phase 3** — Authentication and role-based authorization
- [ ] **Phase 4** — Student profile and dashboard shell
- [ ] **Phase 5** — Task workflow (admin CRUD, student checklist, completion)
- [ ] **Phase 6** — Gamification engine (XP, levels, badges, streaks)
- [ ] **Phase 7** — Dashboard and leaderboard
- [ ] **Phase 8** — Mentoring (requests, accept/reject)
- [ ] **Phase 9** — Real-time chat (Socket.IO)
- [ ] **Phase 10** — Admin operations console
- [ ] **Phase 11** — Testing and hardening
- [ ] **Phase 12** — Deployment and documentation

### First two sprints (backlog priority)
1. Repo, environments, CI, migrations, seed data
2. Auth, email verification, role middleware
3. Admin task CRUD + student task checklist/completion
4. XP ledger, level rule, dashboard progress

## Open Decisions

A few product rules need to be confirmed before/during implementation (full list in SRS Section 3):

- Single college deployment vs. multi-tenant
- Whether task completion requires evidence upload or mentor approval
- Official time zone and "streak grace day" policy
- XP curve and maximum level
- One mentor per student vs. multiple

## Getting Started

*(Setup instructions will be added once Phase 1 — project scaffolding — is complete.)*

```bash
git clone https://github.com/Hamphreykoley/gamified-learning-app.git
cd gamified-learning-app
# frontend and backend setup instructions coming soon
```

## Documentation

Full requirements, data model, API design, business rules, and the 12-phase build plan are documented in the [Software Requirements Specification](./docs/Gamified_Learning_App_SRS.pdf).

## License

All rights reserved. This project is not currently licensed for reuse or distribution.

## Author

**Hamphrey Koley**
B.Tech Information Technology, Netaji Subhash Engineering College
