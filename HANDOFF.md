# HANDOFF — onduty-incident-report

## What it does

On-duty employee incident report app for North Country EMS / Clark Fire District 13. Crew members use it to report injury/illness, biological exposure, chemical exposure, or vehicle incidents. It's a single-file HTML/CSS/JS app (`index.html`, ~756 lines), no build step, meant to run as a home-screen web app on iOS.

Four incident types, each with its own flow:

- **Injury / Illness** — nature-of-injury checklist (burn, head injury, fracture, laceration, stress, etc.)
- **Biological Exposure** — walks through a bloodborne and airborne decision tree (a series of yes/no questions) that classifies the event as contamination, exposure, or safe, and shows the right next step (e.g. "notify your BC immediately" for a true exposure)
- **Chemical Exposure** — chemical/substance name plus a set of scene-condition questions
- **Vehicle Incident** — a separate quick-assessment flow with its own submit and JSON export

Submitted reports live in a Reports tab, gated by a 4-digit PIN, with filter chips by type (Injury/Biological/Chemical/Vehicle) and per-report actions (view, mark reviewed, close, export PDF).

## Where the data lives

- **Reports themselves: Doug's Cloudflare Worker + KV** (as of v2.9, 2026-08-07 — replaced Derek's jsonbin.io). GET/PUT against `https://ncems-api.d-boyce.workers.dev/bin/incident-reports`. Reads are open; writes require the `X-App-Key` header. The Worker URL and app key defaults are hardcoded in the JS as `DEFAULTS` (`workerUrl`, `appKey`). See `CLOUDFLARE_SETUP.md` in the parent Github folder for the full Worker/KV setup.
- **App settings (Worker URL, app key, notify email, EmailJS creds, reports PIN, Google Maps key): browser `localStorage`**, key `ncems_ir_v2`, JSON blob. Falls back to `DEFAULTS` if empty or unparseable. Settings are per-device. Note: devices with settings saved under the old jsonbin version stored `binId`/`apiKey` keys — the new code ignores those and falls back to the Worker defaults, so no cleanup is needed.
- Default reports-access PIN is `1234`. Default notify email is `d.langdon@northcountryems.org`.
- EmailJS fields exist for optional email notifications on submit, but ship blank — email alerts are off unless someone fills in a service/template/public key.

## Structure

- Single `index.html` — all CSS in a `<style>` block, all JS inline at the bottom, no external JS files.
- Tab-based nav (`.tab-page` / `.nav-btn`) — Home/incident-type picker, the active report form, Reports list, Settings.
- `showTab(t)` drives tab switching and lazy-loads settings form / reports list when you land on those tabs.
- Biological exposure logic is a small rules engine: `bbeA`/`abeA` arrays of yes/no answers checked with `allAns()`/`anyYes()`, producing one of three `dt-result` states (contamination / exposure / safe) with canned guidance text.

## Anything odd

- **README.md is empty** (literally a blank file with one space) — no doc existed before this handoff.
- The app key ships in the page source, same pattern as the old jsonbin key — but unlike the old jsonbin master key it only allows writes to bins on Doug's Worker, and it can be rotated in seconds (set a new `APP_KEY` secret on the Worker, update `DEFAULTS.appKey`, redeploy). Still: no real auth beyond a 4-digit PIN, **not HIPAA/PHI-safe**.
- Settings changes are local-only per device, same caveat as WAMBOchecker's admin mode — there's no push-to-all-devices mechanism, so if someone edits the Worker URL or PIN on one phone, every other phone keeps its own values until manually updated.
- Vehicle Incident has its own separate submit/export path (`submitVehicleReport()`, `exportVehJSON()`) rather than funneling through the same flow as the other three incident types — worth knowing if you're modifying the common report pipeline, since Vehicle may not pick up changes made to the others.
