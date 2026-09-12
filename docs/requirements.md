# TutorFlow — Requirements (User Stories)

## Platform Admin

**PA-1** — As a Platform Admin, I want to add a new institution to the platform, because the product will eventually be sold to other tutoring centers.
Acceptance criteria: creating an institution also creates its first Owner account; the institution starts in an active state; the action is written to the audit log.

**PA-2** — As a Platform Admin, I want to deactivate an institution, because a center might stop using the product or fall behind on payment terms.
Acceptance criteria: all users under that institution lose access immediately; existing appointment data is preserved, not deleted; the action is audit-logged.

**PA-3** — As a Platform Admin, I want to deactivate or reactivate an Owner account, because ownership disputes or account issues need a resolution path that doesn't sit with the institution itself.
Acceptance criteria: only the Platform Admin can perform this — no Director or Owner has this permission over another Owner.

**PA-4** — As a Platform Admin, I want to view the audit log across every institution, because I'm responsible for the platform as a whole, not one tenant.
Acceptance criteria: log entries include actor, action, target, institution, and timestamp; viewing the log is itself audit-logged.

---

## Owner

**OW-1** — As an Owner, I want to configure institution-level settings (lesson duration, working days, holidays, booking horizon), because each tutoring center runs on its own schedule.
Acceptance criteria: settings apply only to my own institution; changing lesson duration doesn't retroactively alter already-booked appointments.

**OW-2** — As an Owner, I want to assign the Director role to a user, because I can't run daily operations myself.
Acceptance criteria: I can only assign roles below my own rank; a Director I create cannot assign another Director (hierarchy is enforced server-side, not just hidden in the UI).

**OW-3** — As an Owner, I want to view my institution's reports and exports, because I need visibility without doing the day-to-day work myself.
Acceptance criteria: export includes PDF and Excel formats; data is scoped to my institution only.

---

## Director

**DR-1** — As a Director, I want to assign Counselor and Teacher roles, because I manage staff below my rank.
Acceptance criteria: I cannot assign another Director; attempting to do so is rejected server-side, not just hidden client-side.

**DR-2** — As a Director, I want to enter and edit a teacher's schedule, because teachers can't edit their own schedule by design.
Acceptance criteria: a teacher working at two institutions can't be scheduled at both on the same day — the system blocks the second entry at save time with a clear message naming the conflicting institution.

**DR-3** — As a Director, I want to approve or reject a pending student registration, because I need to control who enters the system, alongside Counselors who can do the same.
Acceptance criteria: rejection requires a reason; approval activates the student account immediately.

**DR-4** — As a Director, I want inactive students to be deactivated automatically at academic year rollover rather than deleted, because I still need their historical appointment counts in reports.
Acceptance criteria: deactivated students don't appear in active rosters or booking flows; their past appointments still count toward institution statistics.

**DR-5** — As a Director, I want to suspend a Counselor or Teacher, because performance or conduct issues come up.
Acceptance criteria: I can only suspend roles below my own rank; the suspended user loses access immediately; the action is audit-logged.

---

## Counselor

**CO-1** — As a Counselor, I want to approve or reject a pending student registration, because gatekeeping who joins the institution is part of my role.
Acceptance criteria: same as DR-3 — rejection requires a reason, approval is immediate.

**CO-2** — As a Counselor, I want to enter and edit a teacher's schedule (free/busy, blocked slots), because teachers don't manage their own calendars in this system.
Acceptance criteria: identical conflict rules to DR-2 — day-level cross-institution conflicts are blocked at save time.

**CO-3** — As a Counselor, I want to add or remove a Teacher or Student, because that's within my rank in the hierarchy.
Acceptance criteria: I can't touch a Director, another Counselor, or anyone above my rank — attempting to returns a permission error, not a silently ignored request.

**CO-4** — As a Counselor, I want to apply a no-show penalty to a student, because repeated no-shows need a consequence.
Acceptance criteria: the penalty has a timeout duration; there's no scheduled job clearing it — the check happens when the student next tries to book, and an expired penalty no longer blocks booking.

**CO-5** — As a Counselor, I want to see which students are currently under a penalty timeout, because I sometimes need to explain or override a block in person.
Acceptance criteria: the list shows remaining timeout duration per student.

---

## Teacher

**TC-1** — As a Teacher, I want to see my own schedule and upcoming appointments, because I need to know who's coming and when.
Acceptance criteria: I only see my own bookings, not other teachers' schedules.

**TC-2** — As a Teacher, I should not be able to edit my own schedule, because that responsibility belongs to Counselors and Directors by design in this system.
Acceptance criteria: no schedule-editing UI or endpoint is reachable from a Teacher account; attempting the underlying request directly returns a permission error.

**TC-3** — As a Teacher, I want to export my own schedule to PDF or Excel, because I sometimes need it outside the platform.
Acceptance criteria: export is scoped to my own schedule only — this is an ownership check, not a role check, since a Director can export any teacher's schedule but a Teacher can only export their own.

---

## Student

**ST-1** — As a Student, I want to self-register, because I shouldn't need staff to create my account for me.
Acceptance criteria: my account stays in a pending state until a Counselor or Director approves it; I can't book anything while pending.
**Mobile note:** the registration form should fit a single scrollable screen without horizontal scrolling — no wide side-by-side field layout.

**ST-2** — As a Student, I want to see a teacher's free slots and book an appointment, because I want to manage my own schedule without calling the institution.
Acceptance criteria: past time slots don't show; holidays don't show; if I've hit my weekly appointment limit, the booking button is disabled and states why (e.g. "You've used 3/3 appointments this week").
**Mobile note:** the teacher list renders as cards, not a table; slot selection opens as a full-screen bottom sheet rather than a side-by-side desktop grid.

**ST-3** — As a Student, I want to cancel my own appointment, because plans change.
Acceptance criteria: cancellation respects any institution-configured cutoff window; I can't cancel someone else's appointment.
**Mobile note:** the cancel action is reachable with one tap from the appointment list, not buried in a submenu.

**ST-4** — As a Student, I want to only see subjects and teachers relevant to my school level (middle school or high school), because a middle schooler shouldn't be booking a high-school-only subject.
Acceptance criteria: the split is enforced at the Subject level; graduate students see the high-school subject set but with weekday-morning scheduling instead of evening/weekend.
**Mobile note:** the level filter (middle school / high school) is visible without needing to open a separate filter screen.

**ST-5** — As a Student, I want to receive an email when my appointment is booked, cancelled, or when I'm penalized, because I don't want to check the platform constantly to find out.
Acceptance criteria: each of the three events triggers exactly one email; email is for notification only — it isn't used for account verification.

**ST-6** — As a Student, I want my past appointment history to remain visible even if I stop attending years later, because the reports built from that data shouldn't lose accuracy on my account.
Acceptance criteria: my account becomes inactive rather than deleted; long-term, it's anonymized rather than deleted, but appointment counts tied to it still count in institution-level reports.