# TutorFlow — Scope (v1)

This document freezes what v1 includes and what it deliberately excludes. If a feature isn't listed under "In scope," it doesn't get built until this document is revised on purpose — not added mid-phase because it seemed convenient.

## In scope (v1)

- Multi-tenant architecture: multiple institutions (kurum) on one platform, data isolated per tenant
- Roles: student, teacher, guidance counselor, course director, course owner/founder, plus a single platform admin above all tenants
- Hierarchical permission model: an actor can only manage roles below their own rank; a director cannot assign another director
- Student self-registration, pending approval by counselor or director
- Appointment booking and cancellation, with conflict checking
- Teacher availability engine (free/busy, blocked slots), same logic as [TutorSchedule](https://github.com/ardaozkaranfil/tutor-schedule-app)
- Institution-level configuration: lesson durations, working days, holidays, per-institution booking horizon
- Middle school / high school split at the Subject level; graduate students stay in the high-school tier, distinguished only by schedule type (weekday morning vs. evening/weekend)
- Teachers can work across multiple institutions (e.g., Monday at one branch, Tuesday at another); day-level conflict prevention across institutions
- No-show penalty system with timeout, checked on read (no scheduled job)
- Audit log for approvals, suspensions, and other sensitive actions
- Email notifications (booking confirmation, cancellation, penalty, etc.)
- Membership lifecycle: inactive users are deactivated, not deleted; long-term, anonymized rather than deleted, so historical appointment counts survive in reports
- Academic-year based rollover (no semester concept)
- PDF/Excel export for schedules and reports
- Platform admin panel: add/remove institutions, manage founder/institution activation — restricted to a single platform admin account
- JWT-based stateless auth
- Mobile-friendly web UI (not a native app)
- Weekly pg_dump backup archive as the primary safety net

## Out of scope (v1) — and why

- **SMS notifications** — email covers the notification requirement for v1; SMS adds a paid provider dependency with no clear ROI yet
- **Parent login / parent panel** — v1 scope is student- and institution-facing; a parent role needs its own permission and data model, deferred to v2
- **Native mobile app** — the web UI is mobile-responsive, which covers the "most users are on their phone" requirement without the overhead of a separate app
- **Payments / billing** — no monetization flow exists yet; institutions are onboarded manually
- **Multi-language (i18n)** — single-language (Turkish) UI is enough for the current two institutions
- **Graphical/dashboard-style reports** — v1 reporting is export-based (PDF/Excel); visual dashboards are a v2 nice-to-have
- **Teachers editing their own schedule** — only counselors and directors can enter/edit the teaching schedule, by design
- **Automatic cross-institution scheduling conflict detection** — only day-level overlap is checked (a teacher can't be marked working at two institutions the same day); slot-level cross-institution conflict detection is not built
- **2FA** — noted for the platform admin account specifically as a v2 backlog item; not required for tenant users in v1

## Notes

- This list reflects decisions confirmed with the institution founder as of the design phase.
- Anything not in the "In scope" list needs a deliberate revision of this document before it gets picked up — not a mid-phase addition.