# NCEMS During Duty Incident Report

Live: https://reports.northcountryems.org/
Upload: https://github.com/emaildougb/onduty-incident-report/upload/main

On-duty incident reporting for North Country EMS, Clark Fire District 13 and the Volcano Rescue Team (VRT): injury or illness, biological exposure, chemical exposure, and vehicle incidents.

**Version 3.3** (10/24/2026)

## Links

- Live: [https://reports.northcountryems.org/](https://reports.northcountryems.org/)
- Repo: [https://github.com/emaildougb/onduty-incident-report](https://github.com/emaildougb/onduty-incident-report)
- Upload: [https://github.com/emaildougb/onduty-incident-report/upload/main](https://github.com/emaildougb/onduty-incident-report/upload/main)
- Storage (Cloudflare Worker): [https://dash.cloudflare.com/](https://dash.cloudflare.com/) > Workers > `ncems-api`

## Files (upload ALL of these)

- `index.html`: the whole app
- `README.md`: this file
- `HANDOFF.md`: notes for whoever works on it next
- `logo.png`, `apple-touch-icon.png`, `icon-512.png`, `manifest.json`: logo and icons (new in 3.3)
- `worker.js`: NOT uploaded to GitHub. This is pasted into the Cloudflare Worker (see WORKER_SETUP.md)

## Page layout

**Top bar:** the standard NCEMS bar (same as CG Staff). Home button goes to the NCEMS Hub. The i button opens details, and the CG Staff section in there holds Reports (and Settings once signed in).
**Step 1:** pick the agency (NCEMS, FD13 or VRT). No bottom bar on this page.
**Step 2:** pick the type: Injury / Illness, Biological Exposure, Chemical Exposure or Vehicle Incident. Vehicle goes to the vehicle report; the others ask the hospitalization question, then open the form with that type already picked.
**Bottom bar (after step 1):** Back (one page at a time) and Start Over. Crews never sign in.
**Reports (under the i):** Sign in with Microsoft. Only the chiefs on the Worker's ADMINS list can see reports. View, mark reviewed, close, add BC and HSO comments, export PDF.
**Settings (under the i, after sign in):** Worker address and a connection test.

## Who can see reports

The Worker checks the Microsoft sign-in and only answers for these emails (change with the `ADMINS` variable in Cloudflare):
Doug Boyce, Derek Langdon, Dennis Madler, Bryce Shirley, Chuck Andrus.

## Where the data lives

- Cloudflare Worker `ncems-api.d-boyce.workers.dev`, Doug's Cloudflare account, KV storage.
- `POST /bin/incident-reports/report`: crews add one report. They can't read, change or delete. Limit 10 reports per device per hour.
- `GET` and `PUT /bin/incident-reports`: chiefs only, with a Microsoft sign-in token.
- A backup copy is saved once a day before any chief edit and kept 30 days.

## Microsoft sign-in setup

1. Uses the existing Entra app "NCEMS Grounds Tracker" (client id `254c214f-bfe2-4ebb-9294-99815e167ab0`)
2. Entra > App registrations > NCEMS Grounds Tracker > Authentication > Single-page application > add `https://reports.northcountryems.org/`
3. Save. No new permissions needed (it only asks for sign-in)

## Changelog

- **3.3** (10/24/2026): Renamed to During Duty Incident Report. Standard NCEMS top bar like CG Staff (home to the Hub, logo, i button). Tab icon added (apple-touch-icon.png, icon-512.png, manifest.json, logo.png; the browser tab showed a blank globe before). New order: agency, then type of incident (vehicle is now one of the types), then the questions. No bottom bar on the first page; after that the bar is Back and Start Over. Reports and Settings moved under the i (CG Staff section). Haptics on taps. iPhone status bar set to default so the clock shows on the white bar

### v3.2
- Step 1 now has a third agency button, VRT, next to NCEMS and FD13 (orange, so it stands out from the other two)
- VRT reports get their own reference numbers: VRT-YYYYMMDD-#### and VRT-VEH-YYYYMMDD-#### for vehicle incidents
- VRT contact list on the call screens: Chief Shirley and BC Boyce (VRT Incident Commander) up top, then DC Andrus, BC Langdon, BC Madler
- Agency shows as VRT on the report, the chiefs' list, the detail view and the PDF
- Version line in Settings fixed (it still said 2.9)
- No Worker change expected (the app sends agency as plain text). Send one test VRT report after uploading to confirm the Worker saves it

### v3.1
- New phone icon (10/17)

### v3.0
- Security fix from the October 2026 audit
- Reports screen now uses Microsoft sign-in instead of the 4 digit PIN
- Crews submit with "add one report" only. The public write key is gone from the code
- Everything shown in the reports list, detail view and PDF is cleaned first, so planted code can't run
- Needs the new Cloudflare Worker (`worker.js`) in place at the same time

### v2.9
- Moved storage from jsonbin.io to Doug's Cloudflare Worker

## Notes

- iOS home-screen caches hard. After this update, crews should delete and re-add the icon, or an old copy may fail to submit
- Hosted at reports.northcountryems.org (CNAME to emaildougb.github.io)
- Not for patient information. Employee incident data only, chiefs-only access
