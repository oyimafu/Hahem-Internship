# Defender Portal Collection Runbook — Client Session

**Purpose:** collect every figure and artefact the audit evidence report needs from Microsoft Defender for Office 365 Attack simulation training, scoped to the users each payload actually targeted.
**Payloads:** FIN-01 · SIGNIN-01 · SC-01
**Fill target:** `mubadala_payload_evidence_template.csv` (one row per payload) and the artefact table in each dossier
**Time:** allow 60–90 minutes with the client, plus 30 minutes post-session reconciliation

---

## 0. Agree the denominators before touching a number

Write these on the whiteboard first. Every rate in the report depends on them and the auditor will ask.

| Term | Definition to agree | Where it comes from |
|------|--------------------|---------------------|
| **In-scope population** (coverage denominator) | Entra ID users with `userType = Member`, `accountEnabled = true`, holding an Exchange licence, **excluding** guests, shared / resource / room mailboxes, service accounts and any agreed exclusions (e.g. executive protection group) | Entra ID export, not Defender |
| **Targeted** (per payload) | Rows in the simulation's Users export **after removing** any guest / external / out-of-scope account that was swept into the target group | Simulation → Users tab → Export |
| **Delivered** (rate denominator) | Targeted rows where *Simulation message delivery* = Successfully received (`SuccessfullyDeliveredEmail_TimeStamp` populated) | Same export |
| **Compromised** | `Compromised = Yes`. Credential Harvest → credentials entered. Drive-by URL → link clicked | Same export |
| **Reported** | `Phishing Reported On` populated (Report Phish add-in only; non-Microsoft reporting tools are not captured) | Same export |

**Why the external-user problem exists.** Guests (`#EXT#` in the UPN, `userType = Guest`) and shared mailboxes sit in the tenant directory. If a simulation targeted "all users" or a group that contains them, they appear in the Users export; and the org-wide *User coverage* card counts all of them as non-simulated. Both distort the figures unless you filter.

---

## 1. Permissions and setup (before the session)

- Account with **Attack Simulation Administrator** or **Attack Payload Author** + **Security Reader** (or Global Reader) in the Defender portal.
- For the Entra denominator: **Global Reader** or `User.Read.All` via Graph PowerShell.
- For the audit-log evidence: **Audit Reader** / Purview Audit access.
- Have a clean folder ready: `Evidence/<YYYYMMDD>/` with sub-folders `FIN-01`, `SIGNIN-01`, `SC-01`, `Programme`.
- Screenshot tool that shows the system clock; name files `E-05_<PayloadID>_<screen>_<YYYYMMDD>.png`.
- Excel or PowerShell for the post-session filtering.

---

## 2. Session flow — per payload (repeat for FIN-01, SIGNIN-01, SC-01)

Portal: `security.microsoft.com` → **Email & collaboration → Attack simulation training → Simulations** tab.
Direct link: <https://security.microsoft.com/attacksimulator?viewid=simulations>

### 2.1 Details tab — identity (screenshot each section)

| Capture | Feeds |
|---------|-------|
| Simulation name, status, **simulation ID / URL** from the address bar | Identification |
| **Technique** (Credential Harvest / Drive-by URL / …) — this resolves the `[confirm]` in the report | Identification; changes the compromise definition |
| Payload name; login page name (Credential Harvest only); landing page name | Identification; Artefacts |
| **Target users** — the included users / groups and the excluded users / groups. Note whether "all users" or a group was used | Population; external-user filtering |
| Launch date and time; end date; delivery schedule (all at once / staggered / region-aware) | Identification |
| Training assignment: modules, trigger (all / clicked / compromised), **due date**, notification templates | Training |

### 2.2 Report tab — headline cards (screenshot the whole tab, then each card)

| Card | Fields to record |
|------|------------------|
| **Simulation impact** | Compromised users (count and %); Users who reported (count and %) |
| **All user activity** | Clicked message link (or Attachment opened / link clicked); Supplied credentials; Read message; Deleted message; Replied to message; Forwarded message; Out of office |
| **Delivery status** | Successfully received message; Failed to deliver; Positive reinforcement delivered; Just simulation message delivered; **Excluded users or groups** (open the flyout, screenshot the list) |
| **Training completion** | Per module: assigned, completed, in progress, not completed |
| **First & average instance** | First link clicked; Avg link clicked; First credential entered; Avg credential entered (times after launch) |

Record the portal figures as a cross-check only — the report's numbers come from the filtered export in 2.3.

### 2.3 Users tab — the export that becomes the evidence

1. Select **Customize columns** and turn on **every** column.
2. Select **Export report** → save as `E-01_<PayloadID>_Users_<YYYYMMDD>.csv`.
3. Do **not** apply portal filters before exporting — filter afterwards so the raw export is complete.
4. While on the tab, apply *Compromised: Yes* and screenshot the count; then *Reported message: Yes* and screenshot; then *Simulation message delivery: Failed to deliver* and screenshot. These three screenshots reconcile against the export.

Columns you will use from the export:

`UserName` · `UserMail` · `Department` · `Title` · `Office` · `Manager` · `Compromised` · `SuccessfullyDeliveredEmail_TimeStamp` · `FailedToDeliverEmail_TimeStamp` · `MessageRead_TimeStamp` · `EmailLinkClicked_TimeStamp` · `EmailLinkClicked_ClickSource` · `CredSupplied_TimeStamp(Compromised)` · `Phishing Reported On` · `MessageDeleted_TimeStamp` · `MessageReplied_TimeStamp` · `MessageForwarded_TimeStamp` · `OutOfOfficeDays` · `TrainingAssignmentMessageDelivered_TimeStamp` · `Assigned Trainings` · `Completed Trainings` · `Training Status`

### 2.4 Content library — the payload record and templates

Direct link: <https://security.microsoft.com/attacksimulator?viewid=contentlibrary>

| Location | Capture |
|----------|---------|
| **Payloads → Tenant payloads → [payload]** flyout, Overview tab | From name; **From email**; **Email subject**; Technique; Theme; Brand; Industry; Language; **Predicted compromise rate**; Click rate; Simulations launched; Tags. Screenshot the rendered preview. |
| Same list, **Filter → Complexity** | Note which band (High / Medium / Low) the payload falls in — this is the *Defender complexity* field |
| Payload flyout → **Simulations launched** tab | Click rate and compromised rate per simulation using this payload |
| **Login pages** (Credential Harvest only) | Name; screenshot the preview |
| **Landing pages** | Name; screenshot the preview |
| **End user notifications** | Names of the positive-reinforcement, training-assignment and training-reminder templates; screenshot each preview |

If the payload was built with **indicators** (Add indicators step), open *Edit payload → Add indicators* read-only and screenshot the list; it corroborates the indicator count in the dossier.

---

## 3. Programme-level collection (once)

Portal: **Reports** tab → **Attack simulation report**. Direct link: <https://security.microsoft.com/attacksimulationreport>

| Tab | Action | Caution |
|-----|--------|---------|
| **Training efficacy** | Export report → `E-02_TrainingEfficacy_<date>.csv`. Gives per simulation: technique, tactics, **predicted vs actual compromise rate**, total users targeted, clicked users | Per simulation, so already scoped. Safe to use directly |
| **User coverage** | Export report → `E-02_UserCoverage_<date>.csv`; screenshot the card for the record only | **Do not use the card percentage.** The denominator is the whole tenant including guests. Recompute: distinct in-scope users with *Included in simulation = Yes* ÷ in-scope population from Entra (section 4) |
| **Training completion** | Export report → `E-02_TrainingCompletion_<date>.csv`; filter *Status* if needed | Filter to in-scope users after export |
| **Repeat offenders** | Export report → `E-02_RepeatOffenders_<date>.csv`; filter by simulation type if useful | Read the **repeat offender threshold** first: **Settings** tab → *Repeat offender threshold* (default 2). Screenshot it |

**Settings tab** (<https://security.microsoft.com/attacksimulator?viewid=setting>): screenshot the repeat offender threshold and any training threshold settings.

**Training tab** (<https://security.microsoft.com/attacksimulator?viewid=trainingcampaign>): if any standalone Training campaigns were run, open each → Report tab → screenshot *Training completion summary*; Users tab → Export.

---

## 4. Supporting evidence outside Attack simulation training

| Evidence | Where | What to capture |
|----------|-------|-----------------|
| **In-scope population** (coverage denominator) | Entra admin centre → Users, or Graph PowerShell (section 6) | Count and list of Member, enabled, licensed users with `Department`; exclude shared / resource mailboxes (Exchange admin centre → Recipients → Mailboxes → filter type) |
| **Guest / external list** (to remove from exports) | Entra → Users → filter *User type = Guest* | Export UPN list |
| **Simulation allow-listing** (proves controls were bypassed only for the simulation) | Email & collaboration → Policies & rules → Threat policies → **Advanced delivery → Phishing simulation** tab (<https://security.microsoft.com/advanceddelivery>) | Screenshot the configured sending domains / IPs / simulation URLs and the last-modified date. Feeds evidence E-06 |
| **Report Phish channel exists** | Settings → Email & collaboration → **User reported settings** (<https://security.microsoft.com/securitysettings/userSubmission>) | Screenshot: reporting enabled, add-in / built-in button, reported-message mailbox |
| **Who launched what, when** | Microsoft Purview → Audit → search **Record types** `AttackSim`, `AttackSimAdmin`, `UserTraining` over the assessment window | Export results → `E-08_AuditLog_<date>.csv` |

---

## 5. Post-session — filtering and reconciliation (per payload)

Work on a **copy** of each Users export; keep the original untouched and hashed.

1. **Remove out-of-scope rows.** Delete rows where `UserMail` domain ≠ `mubadalaenergy.com`, or `UserName` contains `#EXT#`, or the UPN appears in the guest list, or the mailbox is shared / resource / service. Record how many rows were removed and why — that count goes in the report as "excluded from denominator: [X] guest / [X] shared".
2. **Targeted** = remaining rows.
3. **Delivered** = rows with `SuccessfullyDeliveredEmail_TimeStamp` populated. **Failed** = rows with `FailedToDeliverEmail_TimeStamp`. Targeted must equal Delivered + Failed; if not, find the gap before going further.
4. **Read / Clicked / Compromised / Reported / Deleted / Replied / Forwarded / OOO** = count of populated timestamps (or `Compromised = Yes`, `OutOfOfficeDays > 0`).
5. **Reported before click** = `Phishing Reported On` populated **and** (`EmailLinkClicked_TimeStamp` blank **or** later than the report time). **Reported after click** = the remainder of reporters.
6. **Rates** (÷ Delivered): click rate, compromise rate, report rate. **Click → compromise** = Compromised ÷ Clicked. **Resilience ratio** = Reported ÷ Compromised.
7. **Actual − predicted** = compromise rate − the payload's Predicted compromise rate (from 2.4), in percentage points.
8. **Time-to-report** per reporter = `Phishing Reported On` − `SuccessfullyDeliveredEmail_TimeStamp`; take the **median**. Excel: `=MEDIAN(IF(reported<>"", reported-delivered))` as an array formula, or PowerShell.
9. **Training** = counts of `Training Status` values; overdue = incomplete with due date passed.
10. **Department view** = pivot on `Department`: delivered, compromised, reported per department. Note any rows with blank `Department` — that is an Entra data-quality finding for the report.
11. **Reconcile** against the Report-tab screenshots. Differences should equal exactly the rows you removed in step 1; document any other variance.
12. **Hash** every original export: `Get-FileHash -Algorithm SHA256 <file>` and record the hash in the evidence CSV and the dossier artefact table.

---

## 6. Optional — pull the same data by script

Faster and reproducible; still take the portal screenshots for the auditor.

**In-scope denominator (Graph PowerShell)**
```powershell
Connect-MgGraph -Scopes "User.Read.All"
Get-MgUser -All -ConsistencyLevel eventual -CountVariable c `
  -Filter "userType eq 'Member' and accountEnabled eq true" `
  -Property userPrincipalName,mail,department,assignedLicenses |
  Where-Object { $_.AssignedLicenses.Count -gt 0 } |
  Select-Object userPrincipalName,mail,department |
  Export-Csv .\Programme\in_scope_users.csv -NoTypeInformation
# then remove shared/resource mailboxes:
Connect-ExchangeOnline
Get-EXOMailbox -ResultSize Unlimited -RecipientTypeDetails SharedMailbox,RoomMailbox,EquipmentMailbox |
  Select-Object UserPrincipalName | Export-Csv .\Programme\non_user_mailboxes.csv -NoTypeInformation
```

**Per-simulation user report (Graph, `AttackSimulation.Read.All`)**
```powershell
Connect-MgGraph -Scopes "AttackSimulation.Read.All"
$sims = Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/v1.0/security/attackSimulation/simulations"
$sims.value | Select-Object id,displayName,attackTechnique,status,launchDateTime,completionDateTime
$id = "<simulationId>"
Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/v1.0/security/attackSimulation/simulations/$id/report/overview"
Invoke-MgGraphRequest -Method GET -Uri "https://graph.microsoft.com/v1.0/security/attackSimulation/simulations/$id/report/simulationUsers"
```
The `simulationUsers` response carries `isCompromised`, `compromisedDateTime`, `reportedPhishDateTime`, `assignedTrainingsCount`, `completedTrainingsCount`, `inProgressTrainingsCount` and a `simulationEvents` list per user. Validate field names against the tenant's Graph version on first run.

---

## 7. Session checklist (tick as you go)

**Per payload — FIN-01 ☐  SIGNIN-01 ☐  SC-01 ☐**
- ☐ Details tab screenshot: technique, payload, pages, notifications, **target selection**, launch / end, training due date
- ☐ Simulation ID / URL recorded
- ☐ Report tab: full-page screenshot + each card
- ☐ Excluded users / groups flyout screenshot
- ☐ Users tab: all columns on → Export report → CSV saved
- ☐ Users tab screenshots: Compromised = Yes · Reported = Yes · Failed to deliver
- ☐ Payload flyout: from, subject, PCR, complexity band, preview screenshot
- ☐ Login page (if Credential Harvest), landing page, notification previews screenshotted
- ☐ Indicators list screenshotted (if configured)

**Programme**
- ☐ Training efficacy export · ☐ User coverage export (card % not used) · ☐ Training completion export · ☐ Repeat offenders export
- ☐ Settings: repeat offender threshold screenshot
- ☐ Training campaigns (if any): report screenshot + Users export
- ☐ Advanced delivery → Phishing simulation screenshot
- ☐ User reported settings screenshot
- ☐ Purview audit export (AttackSim, AttackSimAdmin, UserTraining)
- ☐ Entra: in-scope population export · guest list export · shared / resource mailbox list

**Post-session**
- ☐ Out-of-scope rows removed and counted per payload
- ☐ Targeted = Delivered + Failed reconciled
- ☐ Rates, time-to-report, resilience ratio, actual − predicted computed
- ☐ Department pivot built; blank departments noted
- ☐ Originals hashed (SHA-256) and recorded
- ☐ `mubadala_payload_evidence_template.csv` and dossier artefact tables filled
