# Olutech Agro Project Management System

This version includes the professional workflow and accountability upgrade.

## Run locally

```bash
npm install
npm start
```

Open: `http://localhost:3000`

## Default first-use logins

The login page no longer displays demo credentials. Users now sign in with their official Olutech Agro email addresses.

- `admin@olutechagro.com / admin123` - Level 5 System Admin / Project Creator
- `onaope@olutechagro.com / olutech123`
- `dipo@olutechagro.com / olutech123`
- `bolade@olutechagro.com / olutech123`
- `chikanma@olutechagro.com / olutech123`
- `waheed@olutechagro.com / olutech123`
- `akin@olutechagro.com / olutech123`
- `amana@olutechagro.com / olutech123`

After logging in, each user can open **My Password** / **Change Password** to set a private password. Temporary passwords should be changed before live company use.

## Login and password rules

- Login now uses official `@olutechagro.com` email addresses instead of short usernames.
- The login page does not show default login details.
- Users can change their own passwords from **My Password** / **Change Password**.
- Admins can still reset employee passwords from the Users module.
- New or reset passwords are marked as temporary until the user changes them.

## Key workflow rules

- Project creation remains restricted to Level 5 only.
- Project editing is restricted to Level 5 only.
- Workflow steps can be added and edited after a project has been saved.
- Only Level 5 can add, edit, reorder, or delete workflow steps.
- Normal task creation starts from Level 2.
- Users below Level 5 only see tasks assigned to them or tasks they assigned.
- Users can only update tasks assigned to them or assigned by them.
- Workflow progress, current owner, current role, next milestone, current blocker, project health, and decision required are automatically recalculated.
- When a workflow task is completed, the next workflow step automatically becomes ready for the next assigned user.

## NXP checklist change

The NXP Checklist is no longer a separate sidebar section. NXP/documentation work is now managed as a task type: `NXP / Documentation Checklist`.

Existing NXP checklist records are automatically migrated into assignable tasks when the app runs.

## Delay and risk automation

The system includes an embedded automation engine:

- A blocked, delayed, or overdue task automatically creates or updates a connected delay record.
- The delay is connected to the relevant project and task.
- An open delay automatically creates or updates a connected risk record.
- When the task is completed/closed, the auto delay and linked auto risk are closed.
- Project health is calculated from overdue tasks, blocked/delayed tasks, open delays, open approvals, and open risks.

This is self-contained rule-based automation. No external AI API key is required.

## Time clock rules

- AKIN is excluded from the Time Clock module.
- Other users can clock in and clock out.
- Users can only see their own time-clock records.
- Level 5 can see all time-clock records, excluding users who are not required to clock in.

## Dashboards

- Level 5 sees the executive dashboard: Weekly Operations Dashboard, Accountability Register, project watchlist, risks, delays, approvals, and live KPIs.
- Other users see their personal workspace dashboard: assigned tasks, ready workflow steps, projects, work hours, files, and clickable management shortcuts where their clearance allows it.


## Login session duration

Login sessions now last for **30 days** by default instead of expiring after a short workday session. The session token is also saved in the same persistent database used by the rest of the system, so Vercel cold starts should not immediately log users out.

You can change the duration with this optional environment variable:

```text
OLUTECH_SESSION_DAYS=30
```

Logging out manually still ends the login session immediately. Clock-in records are separate from login sessions: if a user clocks in and later logs out, the open clock-in remains open until that user manually clocks out.

## Persistence and deployment note

This version adds a durable persistence layer while keeping the core app structure intact.

Locally, the app continues to save records in `storage/db.json` and uploaded files in `storage/uploads/`. These local records remain after logout, server restart, and laptop restart unless the folder is deleted.

For Vercel/serverless deployment, local storage can reset. To keep projects, tasks, Excel imports, uploaded file records, and open clock-in sessions for days or weeks, connect a durable KV/Redis store and set these environment variables in the hosting platform:

```text
KV_REST_API_URL=your_kv_or_upstash_rest_url
KV_REST_API_TOKEN=your_kv_or_upstash_rest_token
OLUTECH_PERSISTENCE_KEY=olutech_pm_system_db
```

When those variables are available, the app saves the full database to durable cloud storage and reads it back on every request. Excel import workbooks and ordinary uploaded files are also stored inside the database as a fallback archive, so downloads can still work even if temporary server files disappear.

Important: for very heavy production file storage, a dedicated object store is still better. This version is designed to solve the immediate reset problem without changing the core system.

## Troubleshooting: Unexpected end of JSON input

If you see `SyntaxError: Unexpected end of JSON input`, it means `storage/db.json` was empty or corrupted. This version now protects against that by rebuilding the local database from `data/seed.json` and saving a `.corrupt-...bak` backup where possible.

For a manual reset, stop the server, delete `storage/db.json`, then run `npm start` again. The app will recreate the database from the seed file.

## Excel tracker integration

This version includes a Level 5-only Excel Import module.

Open `Excel Import` from the sidebar, upload an Olutech tracker workbook, and the system will distribute the workbook data into the correct modules:

- `Setup` refreshes staff/accountable parties and dropdown values.
- `Project Register` creates or updates project records.
- `Master Productivity Tracker` creates or updates task records.
- `Export-NXP Checklist` becomes assignable `NXP / Documentation Checklist` tasks.
- `Action Log` creates or updates action records.
- `Delay & Escalation Tracker` creates or updates active delay records.
- `Executive Approval Tracker` creates or updates approval records.
- `Risk Register` creates or updates active risk records.
- `Lessons Learned` creates or updates lesson records.

Resolved/closed/cancelled delays and risks are not kept in the active registers. If a delay is resolved, the system removes the delay and any risk linked to that delay. If a risk is resolved, the system removes that risk from the active risk register.

Because Excel import reads `.xlsx`, `.xls`, and `.xlsm` files, run `npm install` after extracting this package so the required Excel reader dependency is installed.

## Excel Integration Field Coverage Update

This version keeps the previous workflow, permissions, delay/risk automation, Level 5 Excel import, and resolved delay/risk purge behavior. It also expands Excel import coverage using `OLUTECH AGRO OPERATIONS PRODUCTIVITY TRACKER (4).xlsx` as the latest source structure.

Additional fields now preserved during import include:

- **Project Register:** days to deadline, task totals, tasks completed, overdue tasks, blocked/delayed tasks, open approvals, open risks, current blocker, next milestone, decision required, impact if delayed, last update, notes, and project folder/link.
- **Master Productivity Tracker:** week start, days outstanding, days to due, overdue flag, blocker owner, approval ID, productivity score, update aging, impact if delayed, and evidence type.
- **Delay & Escalation Tracker:** required decision/action, days outstanding, overdue flag, estimated days impact, evidence link, resolution/recovery plan, and closed date.
- **Executive Approval Tracker:** days outstanding, overdue flag, date approved, decision/comment, and evidence link.
- **Risk Register:** related delay/task references where available, risk score, heat map, early warning, mitigation, contingency, resolution, and closed date.
- **Export-NXP Checklist:** workstream, required-by stage, approver/external party, blocker, escalation requirement, evidence link, and notes.
- **Setup:** task statuses, priorities, yes/no values, approval statuses, risk statuses, escalation levels, evidence types, quality ratings, and project statuses.

Every imported record now stores source sheet, source row, original Excel values, and import warnings where required fields are missing. These warnings appear on the Excel Import page after upload so Level 5 users can correct tracker issues that may affect routing, accountability, automation, or reporting.

## Storage Status Health Check

This version adds a Level 5-only `Storage Status` page in the sidebar.

After deploying to Vercel:

1. Log in as `admin@olutechagro.com / admin123`.
2. Open `Storage Status` from the sidebar.
3. Click `Run Storage Test`.
4. Confirm it says `Persistent Storage OK`.

If it says `Temporary Vercel storage - data will reset`, your Vercel deployment is still missing working cloud storage variables. Add the variables to the correct Vercel environment and redeploy.

The app accepts either of these environment variable pairs:

```text
KV_REST_API_URL
KV_REST_API_TOKEN
```

or:

```text
UPSTASH_REDIS_REST_URL
UPSTASH_REDIS_REST_TOKEN
```

Keep this variable as well:

```text
OLUTECH_PERSISTENCE_KEY=olutech_pm_system_db
```

For the live production URL, set the variables under `Production`. For preview URLs, also set them under `Preview`.

## Latest professional updates

This version adds the requested workplace workflow improvements while keeping the existing core system intact.

### Automatic clock-out
- Time-clock users are automatically clocked out at the configured workday close time, default `17:00` Africa/Lagos.
- Manual clock-out still works before 5pm.
- Auto clock-out records are marked as `Auto Clocked Out` and remain saved in persistent storage.
- AKIN remains excluded from time-clock tracking.

### Task management improvements
- Tasks are separated into **Open Tasks** and **Closed Tasks** on the Tasks page.
- Closed tasks remain visible for audit, dashboard reporting, and accountability.
- Dashboards now show closed/completed task counts.
- Task forms now include a **Task Comment / Work Notes** field.
- Task chat remains persistent after logout and after task completion.

### Project creation improvements
- Level 5 users can create two project types:
  - **Workflow Project**: sequential role-based handoff steps.
  - **No Workflow Project**: standard project that can receive normal assigned tasks.
- Existing workflow settings remain available for workflow projects.
- Level 5 can still add workflow steps to a project later.

### Professional chat UI
- Project and task chat boxes have been restyled to feel more like a Microsoft Teams-style conversation thread.
- Chat history is stored in the database and does not clear after logout, completion, or closure.

## Latest user/login update

The visible demo login details have been removed from the login screen. Users now log in with their official Olutech Agro emails:

- admin@olutechagro.com / admin123
- onaope@olutechagro.com / olutech123
- dipo@olutechagro.com / olutech123
- bola@olutechagro.com / olutech123
- chika@olutechagro.com / olutech123
- akin@olutechagro.com / olutech123

Waheed and Amana have been removed from active user accounts and staff dropdowns. Project and task create panels are collapsed by default. Task pages now support `/tasks?view=open` and `/tasks?view=closed`, so dashboard links can open only the relevant task list.

## Latest password-management update

This version refines password management without changing the core workflow system.

- Normal users only see `Change password` while their account is still using a temporary password.
- After a normal user changes their password, the password menu is removed from their portal.
- Level 5 users keep `Password Admin` access in the top bar/sidebar.
- Level 5 users can change/reset other users' passwords from `Users → Edit user → Change / Reset Password`.
- When a Level 5 user resets another user's password, that user is required to change the temporary password at next login.
- Resetting a user's password also clears that user's active sessions for security.

## Employee management update

Level 5 users can now add completely new employees from **Users → Add new employee / user**. The form asks for display name, official `@olutechagro.com` email address, role, clearance level, account access status, and a temporary password.

New employees are saved into the system staff list and appear as type-to-search suggestions across project owners, workflow assignees, task assignees, delays, approvals, risks, lessons, and other responsibility fields. These fields are no longer limited to fixed dropdown-only staff names.
