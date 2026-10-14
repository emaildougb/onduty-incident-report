# HANDOFF: onduty-incident-report

## What it does

On-duty employee incident report app for North Country EMS / Clark Fire District 13. Crew members use it to report injury/illness, biological exposure, chemical exposure, or vehicle incidents. It's a single-file HTML/CSS/JS app (`index.html`, ~756 lines), no build step, meant to run as a home-screen web app on iOS.

Four incident types, each with its own flow:

- **Injury / Illness** : nature-of-injury checklist (burn, head injury, fracture, laceration, stress, etc.)
- **Biological Exposure** : walks through a bloodborne and airborne decision tree (a series of yes/no questions) that classifies the event as contamination, exposure, or safe, and shows the right next step (e.g. "notify your BC immediately" for a true exposure)
- **Chemical Exposure** : chemical/substance name plus a set of scene-condition questions
- **Vehicle Incident** : a separate quick-assessment flow with its own submit and JSON export

Submitted reports live in a Reports tab, gated by Microsoft sign-in (v3.0, chiefs on the Worker's ADMINS list only), with filter chips by type (Injury/Biological/Chemical/Vehicle) and per-report actions (view, mark reviewed, close, export PDF).

## Where the data lives

- **Reports: Doug's Cloudflare Worker + KV** at `https://ncems-api.d-boyce.workers.dev/bin/incident-reports` (v3.0, October 2026). The Worker code is `worker.js` in the app folder (not uploaded to GitHub, pasted into Cloudflare).
  - Crews: `POST /bin/incident-reports/report` with `{report:{...}}`. Append only, no key, 10 per IP per hour, review fields stripped, status forced to `new`.
  - Chiefs: `GET` / `PUT /bin/incident-reports` with `Authorization: Bearer <Microsoft ID token>`. The Worker checks the signature against Microsoft's keys, the tenant, the audience (Entra app `254c214f-...`, same as CG Staff) and the ADMINS list. A PUT that would drop reports is refused; a daily backup key `backup:incident-reports:YYYY-MM-DD` is kept 30 days.
  - CG Staff reads the same endpoint with its own sign-in token.
- **App settings: browser `localStorage`**, key `ncems_ir_v2` (Worker URL, notify email, EmailJS fields). The old `appKey` and `pin` settings are no longer used.
- EmailJS fields exist for optional email notifications on submit but ship blank.

## Structure

- Single `index.html` : all CSS in a `<style>` block, all JS inline at the bottom, no external JS files.
- Tab-based nav (`.tab-page` / `.nav-btn`) : Home/incident-type picker, the active report form, Reports list, Settings.
- `showTab(t)` drives tab switching and lazy-loads settings form / reports list when you land on those tabs.
- Biological exposure logic is a small rules engine: `bbeA`/`abeA` arrays of yes/no answers checked with `allAns()`/`anyYes()`, producing one of three `dt-result` states (contamination / exposure / safe) with canned guidance text.

## Anything odd

- README.md was empty before v3.0; it now follows the apps format.
- v3.0 removed the public app key and the PIN. Real access control now lives in the Worker. Still not for patient information.
- Settings changes are local-only per device, same caveat as WAMBOchecker's admin mode : there's no push-to-all-devices mechanism, so if someone edits the Worker URL or PIN on one phone, every other phone keeps its own values until manually updated.
- Vehicle Incident has its own separate submit/export path (`submitVehicleReport()`, `exportVehJSON()`) rather than funneling through the same flow as the other three incident types : worth knowing if you're modifying the common report pipeline, since Vehicle may not pick up changes made to the others.
