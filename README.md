# Smart Roster Management System

A browser-based tool to plan, generate, validate and change monthly rosters for a team spread across brands, processes and pods.

**Live app:** `https://aksharasabde25-alt.github.io/roster-management-system/`

## Manager login

The app opens on a sign-in page.

- Username: `manager`
- Password: `Roster@2026`

The password is stored only as a SHA-256 hash, three wrong attempts lock the form for 30 seconds, and Sign out ends the session. **Important:** this is a front-end gate for a demo. With no backend, anyone who reads the page source can bypass it. A real deployment needs server-side authentication (for example Firebase Auth or a small API with hashed passwords and sessions) and role-based access for pod leads.

## Run it

No build step and no dependencies. The whole app is one file, `index.html`.

- Locally: open `index.html` in a browser.
- Deploy: drag the folder into Netlify Drop, or push to GitHub and enable GitHub Pages (Settings → Pages → deploy from `main`).
- No `.env` is needed because there is no backend or API key.

## Features

- Sample data: 32 employees, 4 pods, 20 brands (16 single-employee, 4 with 2–3 employees), 35 processes, plus a backup (floater) employee.
- Configurable per process: operating calendar (365 days / every day except holidays / off Sundays + holidays), minimum staff, and an optional different minimum on Sundays.
- Configurable per employee: pod, assigned processes, preferred weekly-off day, leaves, comp-offs. Public holidays are set globally.
- Roster generation for any month.
- Views: company, pod, brand, process, employee.
- Conflict list with severity and a suggested fix for each coverage gap.
- Click any cell to edit it; coverage is re-checked immediately.
- "Handle a change" tab: add a leave and either keep the roster (just flag gaps) or regenerate.
- Manage data tab: add, edit, delete employees, processes, brands and pods without touching code.
- Publish is blocked while mandatory coverage errors exist.

## Assumptions

- One weekly off per week (Mon–Sun). Weeks with fewer than 5 days inside the month are skipped.
- Closed days are automatic: a Sunday closed by every process an employee works becomes a weekly off, and a closed holiday shows as H.
- An employee counts towards every process they are assigned to on a day they work.
- One shift per day; no shift timings.
- Pod leads work Monday–Saturday and are not part of brand coverage.
- Leaves and comp-offs are treated as already approved.

## Tell us how you think

**1. How did you model employees, brands, processes, pods and their relationships?**
A pod has a lead. An employee belongs to one pod. A process belongs to a brand (the brand is a field on the process, so a brand exists whenever it has a process) and carries its operating calendar and minimum staff. Employees and processes are linked many-to-many through the employee's list of assigned processes. Leaves and comp-offs are lists of dates on the employee. Holidays are global. Everything is data, so new brands, processes and rules need no code change.

**2. How does the system generate a roster and check constraints?**
For each employee and day it first sets the fixed status: leave, comp-off, holiday, or closed by calendar. Then, week by week, each employee still without an off day gets one. For every candidate day it computes the coverage damage, meaning how many of their processes would drop below minimum staff, and picks the lowest, then the preferred day, then the day with the fewest existing offs (fairness). A separate validator then checks every process on every day: working count versus the required minimum. The validator also runs after every manual edit.

**3. Single-resource versus multi-employee brands?**
They use the same model. A single-resource brand is one employee linked to all of its processes. Because the minimum is 1, that employee must work every operating day, so the weekly off can only fall on a closed day (usually Sunday). Brands that run 365 days with one owner get a backup (floater) employee linked to the same processes. Multi-employee brands share or split processes, and the generator staggers their offs so the minimum holds.

**4. What happens when the only employee for a brand is unavailable on a required day?**
The day becomes L for that employee. The brand's coverage row turns red (0/1) and an error appears in the Conflicts tab for each affected process. The suggested fix names employees assigned to the process or in the same pod who are on a weekly off that day, so the manager can ask one of them to work and give a comp-off. The system does not move anyone by itself.

**5. How does it handle conflicting requirements or insufficient staffing?**
It separates mandatory rules from preferences. Coverage and a weekly off are errors and win over preferences. A preferred off day that couldn't be given is only an info note. Seven or more consecutive working days is a warning. Nothing is hidden: if coverage can't be met, the roster is still generated, the gap is flagged with its date, process and a suggestion, and Publish stays blocked.

**6. What is automated and what is left to the manager?**
Automated: fixed statuses, weekly-off placement, coverage checks, and fix suggestions. Left to the manager: approving leave, resolving gaps (who works, who gets a comp-off), manual edits, relaxing a client rule, and publishing. These involve people and client commitments, so the tool recommends and the manager decides.

**7. If the team and brand count doubled?**
Move the data to a database (e.g. Postgres) with an API and user roles. Replace the greedy generator with a constraint solver (e.g. OR-Tools CP-SAT) so it can find rosters the greedy approach misses. Make coverage checks incremental instead of rescanning all employees, regenerate only the affected pod when something changes, and add paging to the views.

**8. One more week?**
First: persistence (backend and database), because data currently resets on refresh. Second: keep manual edits when regenerating. Today regeneration keeps leaves and comp-offs but discards manual weekly-off edits. Then: a solver-based generator, automated tests for the validator, CSV export, an approval workflow for pod leads, and fairness of offs across months.

## Known limitations

- No backend: data lives in the browser session only.
- The generator is greedy, so it may report a gap even when a perfect roster exists.
- A week can be assigned an off that creates 7+ consecutive working days across week boundaries (shown as a warning).
- Suggested fixes only look at people on a weekly off the same day; they don't check skills.
