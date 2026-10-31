# NCEMS During Duty Incident Report

Live: https://reports.northcountryems.org/
Upload: https://github.com/emaildougb/onduty-incident-report/upload/main

On-duty incident reporting for North Country EMS, Clark Fire District 13 and the Volcano Rescue Team (VRT): injury or illness, biological exposure, chemical exposure, and vehicle incidents.

**Version 4.0** (10/31/2026)

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

**Top bar:** the standard NCEMS bar. Home goes to the NCEMS Hub on the start screen; on every other screen it is a Back button. The i button holds Reports and Settings (CG Staff section).
**Start:** agency switch (NCEMS, FD13, VRT), a box showing who gets the report (FD13: Chief Shirley as DICO, DC Andrus and the on duty BC; NCEMS: the on duty BC; VRT: BC Boyce and Chief Shirley), "Which BC is on duty today?" (remembered on the phone until 07:30 the next morning; Not sure sends to all three BCs), the four incident types each with a "What counts?" dropdown, and an unfinished report to resume.
**Injury, biological, chemical:** hospital question (Yes shows who to call), then four steps with a progress bar and Back / Next at the bottom: when and where, the type's questions, what happened, review and send.
**Vehicle:** at the scene card (do not leave the scene, call law enforcement, notify the on duty BC, Policy 327.3), quick questions with the severity bar, the details, review and send.
**Receipt:** reference number with Copy, who has it (emailed or call them), what happens now (FD13 exposures: get with the DICO), save a PDF copy.
**Reports (under the i):** unchanged from 3.3 for now. Moves to CG Staff in a later build.

## Notifications (new in 4.0)

- On send, each person on the list gets an email through the same EmailJS account as the Apparatus Checker (service_uqfd3sj, template_g3fp9a9). The email has the reference, agency, type, who filed it, when and where, an URGENT line when it applies, and a link to the app. No injury or exposure details go in email
- If EmailJS fails, the receipt says so and tells the crew to call
- On duty BC: crews pick it on the start screen (Doug's option 3). `ONDUTY_URL` in index.html is the hook for pulling it from Derek's schedule through ncems-schedule-crews later; empty means the crew picks

## No signal and drafts (new in 4.0)

- If the send fails for lack of signal, the report waits on the phone (localStorage `ir_queue`) and sends on its own when the phone is back online or every minute while the app is open
- Unfinished reports save on the phone as you go and are thrown away after 4 hours, since crews share agency phones. "Throw it away" on the start screen clears one sooner

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

- **4.0** (10/31/2026): Makeover from Doug's mockups (Design canvas https://claude.ai/artifact/C5P6avzhcf2VQecELw5Ngg). Open Shifts look (green, system font, sentence case). Intro splash removed. One start screen with the agency switch, who gets it, the on duty BC pick and "What counts?" dropdowns. Four short steps with a progress bar instead of one long page. The checkboxes for body part, type of injury and activity are now dropdowns in sections, with Left / Right / Both for arms and legs and Add another body part. Small picks (route, type, vehicle details) are tap chips. Vehicle opens with the at the scene card and Policy 327.3. Review screen with Edit per section. New receipt; FD13 exposures say to get with the DICO, who walks them through it (ED, clinic or wait). Emails to the people on the list. Sends later with no signal. Drafts for 4 hours. Report data keeps the same fields, plus unit, hospitalQuestion, notified and appVersion, so the chiefs' list, detail view and PDF work as before

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
