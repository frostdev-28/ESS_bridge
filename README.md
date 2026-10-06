# EasyTime Pro → ESS bridge — go-live kit

Chrome extension that copies biometric punches from **EasyTime Pro 9.0.6** (office network, reached
over the VPN) into the **ESS portal** at `https://champ-hr.com`. The source of truth is
[developer00777/ESS PR #1](https://github.com/developer00777/ESS/pull/1); this repo carries the
version that went live on the first VPN machine, plus one fix made there (see `CHANGES.md`).

| Path | What it is |
|---|---|
| `easytime-bridge/` | The extension. Load this folder in Chrome. Its own `README.md` is the full manual |
| `CHANGES.md` | Changes made during go-live that still need to go back into PR #1 |
| `CLAUDE.md` | Brief for Claude Code. Open Claude in this folder and it picks up where we left off |

## Where things stand (6 Oct 2026)

Done on the first VPN machine:

- [x] PR #1 deployed — `/api/attendance/easytime-import` answers `401 invalid or missing token`
- [x] `EASYTIME_IMPORT_TOKEN` set on the Railway app service (it's in your password manager)
- [x] EasyTime reachable over the VPN; `champ-hr.com` still reachable with the VPN up
- [x] **Test EasyTime** passed: data, `page_size`, newest-first by id **yes**, date filter **works**
- [x] **Test ESS** passed: token accepted as "Railway variable EASYTIME_IMPORT_TOKEN"
- [x] First sync ran and reached *Up to date*
- [x] Fix: the popup now keeps unknown employee codes until they're matched (`CHANGES.md`)

Still open:

- [ ] Confirm the fix works live: the unknown-codes list survives between runs and shows dates
- [ ] **Unknown employee codes.** The first batch had 207 of 330 punches unmatched. The popup lists
      the codes. Find out whether they are missing from ESS profiles or written differently there,
      fix, then **Re-send** the dates shown in the popup
- [ ] Device feed check in ESS (Admin Controls → Biometric → Device feed)
- [ ] Spot-check one employee's day: ESS calendar vs EasyTime
- [ ] Recovery tests: VPN off → red badge, back on → clears; EasyTime logout → amber, login → clears
- [ ] Keep the machine awake, VPN auto-reconnecting, Chrome running in the background
- [ ] Revoke the old handover token; port `CHANGES.md` into PR #1
- [ ] Write `GO-LIVE-REPORT.md` (no tokens or passwords in it)

## Setting up on a new machine

**Only one machine should run the bridge.** Before or right after switching, open
`chrome://extensions` on the old VPN machine and **remove** the bridge there. Two bridges never
double-count (ESS skips duplicates) but they make the Device feed confusing.

1. **Get the code** to `C:\ESS\easytime-bridge-golive` — the same path on every machine, whatever the
   Windows username. Never move or delete it afterwards: Chrome loads the extension from there on
   every start.
   ```powershell
   git clone https://github.com/<your-account>/<repo-name>.git C:\ESS\easytime-bridge-golive
   ```
   No git? Download the ZIP from GitHub (Code → Download ZIP) and extract it so that
   `C:\ESS\easytime-bridge-golive\easytime-bridge\manifest.json` exists.

   Want a different place? Any folder works — Chrome just remembers whatever you pick in step 5. Only
   avoid OneDrive, Desktop or Downloads, which get synced or cleaned up.
2. **Connect the VPN.** Check that both open in Chrome: your EasyTime Pro address and
   `https://champ-hr.com`.
3. **Chrome 144 or newer** (`chrome://version`).
4. In Chrome, **create a profile** just for the bridge (e.g. "ESS bridge") and **log in to EasyTime
   Pro** in it, ideally with a read-only user.
5. Open `chrome://extensions`, turn on **Developer mode** (leave it on), click **Load unpacked** and
   pick `C:\ESS\easytime-bridge-golive\easytime-bridge` (the folder containing `manifest.json`). Pin
   the icon.
6. Chrome Settings → System → turn on **Continue running background apps when Google Chrome is
   closed**.
7. Click the bridge icon → **Settings**:
   - EasyTime Pro address — exactly as you open it, with the port, no trailing slash
   - ESS portal address — `https://champ-hr.com`
   - Import token — paste from your password manager (the same one; never paste it into chat)
   - Leave the rest at the defaults. **Save** and allow access.
8. **Test EasyTime** and **Test ESS** — expect the same results as above.
9. The first sync starts within seconds. It begins from the newest punch ESS already has, so
   nothing is lost from the switch. To be safe, use **Re-send dates** for the days since the old
   machine stopped.

Settings, sync position and the unknown-codes list live in Chrome on each machine; none of it is in
this repo, so step 7 is always needed.

## Updating the extension

After changing any file in `easytime-bridge/`, click the **reload** icon on the bridge in
`chrome://extensions`. Log every change in `CHANGES.md` so it can go back into PR #1.

## Continuing with Claude Code

Open a terminal in the folder and run `claude`:
```powershell
cd C:\ESS\easytime-bridge-golive; claude
```
 It reads `CLAUDE.md` and knows the task, the
rules (it never handles the token or EasyTime passwords) and what is left. Tell it your EasyTime
address when it asks — it is deliberately not stored in this public repo.

## Never commit

The import token, EasyTime or ESS passwords, or screenshots and pastes containing employee names.
This repo is public.
