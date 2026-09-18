# TutorFlow — Role / Permission Matrix

**Status:** Final — basis for the RBAC implementation and authorization tests.

Derived by going through every user story and acceptance criterion in `requirements.md` and `scope.md`. Permission names are written in the exact format they become in code, as an enum (`RESOURCE_ACTION`).

**Legend:** ✅ yes · ❌ no · 🔶 conditional (ownership — own record only, or a restricted subset of ranks, explained in the cell) · — not applicable to this role

**Rank order:** STUDENT / TEACHER (not a manager rank, rank=0) < COUNSELOR (1) < DIRECTOR (2) < OWNER (3) < PLATFORM_ADMIN (institution-independent, top level)

**Rule (A1):** A higher rank inherits all institution-scoped permissions of the ranks below it — Owner ⊇ Director ⊇ Counselor — except for permissions explicitly forbidden to Teacher/Student (e.g. `SCHEDULE_EDIT`, hard ❌ for Teacher per TC-2).

---

## 1. Appointment

| Permission | STUDENT | TEACHER | COUNSELOR | DIRECTOR | OWNER | Source |
|---|---|---|---|---|---|---|
| `APPOINTMENT_CREATE_SELF` | ✅ (self only) | ❌ | ❌ | ❌ | ❌ | ST-2 |
| `APPOINTMENT_CREATE_FOR_STUDENT` | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | Counselor and Director can create an appointment on a student's behalf (e.g. student has no phone, or a broken one) |
| `APPOINTMENT_CANCEL_OWN` | 🔶 own appointment only | ❌ | ❌ | ❌ | ❌ | ST-3 |
| `APPOINTMENT_CANCEL_ANY` | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | Same scenario as CREATE_FOR_STUDENT |
| `APPOINTMENT_VIEW_OWN` | 🔶 own appointments | 🔶 own schedule | — (covered by VIEW_ALL) | — | — | ST-2/ST-3 (student), TC-1 (teacher) |
| `APPOINTMENT_VIEW_ALL` | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | Prerequisite for DR-2/CO-2 (entering schedules) |

## 2. Membership (registration / staffing)

| Permission | STUDENT | TEACHER | COUNSELOR | DIRECTOR | OWNER | Source |
|---|---|---|---|---|---|---|
| `MEMBERSHIP_SELF_REGISTER` | ✅ (pre-auth, public) | — | — | — | — | ST-1 |
| `MEMBERSHIP_LIST_PENDING` | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | Prerequisite for CO-1/DR-3 |
| `MEMBERSHIP_APPROVE` | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | CO-1, DR-3 |
| `MEMBERSHIP_REJECT` | ❌ | ❌ | ✅ (reason required) | ✅ (reason required) | ✅ (A1) | CO-1, DR-3 |
| `MEMBERSHIP_ADD_STUDENT` | ❌ | ❌ | ✅ | ✅ (A1) | ✅ (A1) | CO-3 |
| `MEMBERSHIP_ADD_TEACHER` (= `ROLE_ASSIGN_TEACHER`, single permission) | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | CO-3, DR-1 describe the same action, merged into one permission |
| `MEMBERSHIP_SUSPEND` (staff — Counselor/Teacher/Director account) | ❌ | ❌ | 🔶 TEACHER only (student side is separate — see note) | 🔶 COUNSELOR, TEACHER | 🔶 DIRECTOR, COUNSELOR, TEACHER (full inheritance) | CO-3, DR-5 — full hierarchical inheritance: each rank can suspend every rank below it, never its own rank or above |
| `MEMBERSHIP_REACTIVATE` | ❌ | ❌ | ✅ (A1) | ✅ (A1) | ✅ (A1) | Reactivated on approval during year-end rollover |

**Note — the "remove student" part of CO-3:** the student-facing "remove" in CO-3 is different from staff account suspension (`MEMBERSHIP_SUSPEND`) — a student's no-show penalty (`PENALTY_CREATE`, Section 5) is already covered under Counselor. Fully deactivating a student account (e.g. the student leaves the institution) is a separate action, also covered under CO-3.

**Note — overlap resolved:** the teacher-adding part of DR-1 and CO-3 is treated as a single action.

## 3. Role assignment (hierarchy)

| Permission | STUDENT | TEACHER | COUNSELOR | DIRECTOR | OWNER | Source |
|---|---|---|---|---|---|---|
| `ROLE_ASSIGN_DIRECTOR` | ❌ | ❌ | ❌ | ❌ | ✅ | OW-2 |
| `ROLE_ASSIGN_COUNSELOR` | ❌ | ❌ | ❌ | ✅ | ✅ (A1) | DR-1 |
| `ROLE_ASSIGN_TEACHER` | ❌ | ❌ | ✅ (see Section 2, merged with `MEMBERSHIP_ADD_TEACHER`) | ✅ | ✅ (A1) | DR-1, CO-3 |

**Hard rule:** nobody can assign their own rank or above — Owner cannot assign another Owner, Director cannot assign another Director (server-side enforcement is specifically called out in the OW-2 and DR-1 acceptance criteria).

## 4. Schedule / Timeslot / Institution settings

| Permission | STUDENT | TEACHER | COUNSELOR | DIRECTOR | OWNER | Source |
|---|---|---|---|---|---|---|
| `SCHEDULE_EDIT` (a teacher's free/busy, blocked-slot entries) | ❌ | ❌ **hard rule** (by design, TC-2) | ✅ | ✅ (A1) | ✅ (A1) | CO-2, DR-2, TC-2 |
| `SCHEDULE_VIEW_OWN` | ❌ | 🔶 own schedule only | — | — | — | TC-1 |
| `SCHEDULE_VIEW_ANY` | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | Prerequisite for SCHEDULE_EDIT |
| `TIMESLOT_EDIT` (lesson-slot/duration definitions) | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | OW-1, extended to Director and Counselor |
| `HOLIDAY_EDIT` | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | OW-1, same extension |
| `POLICY_EDIT` (booking horizon / `bookingWindowDays`) | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | OW-1, same extension |
| `INSTITUTION_SETTINGS_VIEW` | ❌ | ❌ | ✅ | ✅ | ✅ | Counselor/Director need this for day-to-day work |

**Critical point (TC-2):** `SCHEDULE_EDIT` being ❌ for Teacher is the **only exception** to A1 inheritance — a deliberate design decision, independent of the rank chain. Flag it as a separate hard-coded rule in code, not something derived from "Teacher's rank is too low" — it is an intentional prohibition.

## 5. Penalty (no-show)

| Permission | STUDENT | TEACHER | COUNSELOR | DIRECTOR | OWNER | Source |
|---|---|---|---|---|---|---|
| `PENALTY_CREATE` | ❌ | ❌ | ✅ | ✅ (A1) | ✅ (A1) | CO-4 |
| `PENALTY_VIEW` (list of students currently under timeout) | ❌ | ❌ | ✅ | ✅ (A1) | ✅ (A1) | CO-5 |
| `PENALTY_REVOKE` (lift a penalty early) | ❌ | ❌ | ✅ | ✅ (A1) | ✅ (A1) | Fully lifts the penalty; the student can book normally again |

## 6. Report / Export

| Permission | STUDENT | TEACHER | COUNSELOR | DIRECTOR | OWNER | Source |
|---|---|---|---|---|---|---|
| `EXPORT_OWN_SCHEDULE` | ❌ | 🔶 own schedule only | — | — | — | TC-3 — **ownership check, not a role check** |
| `EXPORT_ANY_SCHEDULE` | ❌ | ❌ | ✅ | ✅ | ✅ (A1) | TC-3, extended to Counselor |
| `REPORT_VIEW_INSTITUTION` | ❌ | ❌ | ✅ | ✅ | ✅ | OW-3, extended to Counselor |
| `EXPORT_INSTITUTION_REPORT` (PDF/Excel) | ❌ | ❌ | ✅ | ✅ | ✅ | OW-3, extended to Counselor |

## 7. Audit log (institution-internal)

| Permission | STUDENT | TEACHER | COUNSELOR | DIRECTOR | OWNER | Source |
|---|---|---|---|---|---|---|
| `AUDIT_LOG_VIEW_INSTITUTION` | ❌ | ❌ | ❌ | ❌ | ❌ | Nobody at institution level can read the audit log — only Platform Admin can (`AUDIT_LOG_VIEW_ALL`, Section 8). This permission does not exist in the institution-scoped matrix; it only exists at platform level. |

---

## 8. Platform-level permissions (institution-independent, PLATFORM_ADMIN only)

| Permission | PLATFORM_ADMIN | Source |
|---|---|---|
| `INSTITUTION_CREATE` | ✅ | PA-1 |
| `INSTITUTION_DEACTIVATE` | ✅ | PA-2 |
| `INSTITUTION_REACTIVATE` | ✅ | Implied by scope.md's "manage founder/institution activation" |
| `OWNER_ACCOUNT_DEACTIVATE` | ✅ | PA-3 |
| `OWNER_ACCOUNT_REACTIVATE` | ✅ | PA-3 ("deactivate or reactivate") |
| `AUDIT_LOG_VIEW_ALL` (across all institutions) | ✅ | PA-4 |

---

## 9. Suspend / Assign hierarchy matrix (who can manage whom)

| Actor | Can assign | Can suspend | Cannot assign/suspend |
|---|---|---|---|
| PLATFORM_ADMIN | — (does not assign roles; manages institution/owner activation) | OWNER, INSTITUTION (any) | its own rank (single platform admin, no peer rank to manage) |
| OWNER | DIRECTOR | DIRECTOR, COUNSELOR, TEACHER (full inheritance) | another OWNER, PLATFORM_ADMIN |
| DIRECTOR | COUNSELOR, TEACHER | COUNSELOR, TEACHER | DIRECTOR, OWNER, PLATFORM_ADMIN |
| COUNSELOR | — (no role assignment, only "add/remove" — see Section 2) | TEACHER, STUDENT | COUNSELOR, DIRECTOR, OWNER, PLATFORM_ADMIN |
| TEACHER / STUDENT | — | — | everyone |

Rule: `canManage(actorRank, targetRank) = actorRank > targetRank`. Every assign and suspend operation must go through this single function — an attempt on a rank above the actor's must return a permission error, not be silently ignored (CO-3 acceptance criteria).