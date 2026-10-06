# Task: finish go-live of the EasyTime Pro → ESS bridge

You are Claude Code on a machine that runs the bridge. Read `README.md` first: its *Where things
stand* checklist is the current state, and the open items are your job. The full original brief
(rules, steps, troubleshooting) was in the handoff from session `4b29e270-4569-481b-b45d-29b24eb363f8`;
the essentials are repeated here.

## Context

- **ESS** = Champ HR self-service portal, `https://champ-hr.com` (Railway, behind Cloudflare). Repo
  https://github.com/developer00777/ESS, work is **PR #1** (branch `easytime-chrome-bridge`,
  extension lives at `integrations/easytime-bridge/` there).
- **EasyTime Pro 9.0.6** (ZKTeco) is reachable only over the VPN. Ask the user for its address; it is
  intentionally not in this public repo.
- **The bridge** is `easytime-bridge/`, loaded unpacked in a dedicated Chrome profile. It reads
  `GET <easytime>/iclock/api/transactions/` with the browser's EasyTime login and POSTs to
  `https://champ-hr.com/api/attendance/easytime-import` with the import token
  (`EASYTIME_IMPORT_TOKEN` on the Railway app service).
- On the first VPN machine everything up to the first sync passed; see `README.md`.

## Rules

- **Never ask for, print, store or echo passwords or the import token.** The user types the token
  into the extension's settings. Do not read Chrome's extension storage files: the token is in there.
- **Never send credentialed requests to EasyTime or log in to it.** The user does EasyTime logins.
- **No test POSTs to production ESS.** Read-only `GET` checks only.
- **No changes to the production database or Railway settings.** Tell the user what to change.
- You cannot click in Chrome: give exact click-steps and ask the user to paste back what it shows
  (with names blanked).
- **Code changes:** edit `easytime-bridge/`, have the user click reload on `chrome://extensions`, and
  log every change in `CHANGES.md` — they must go back into PR #1.
- **This repo is public.** Never commit tokens, passwords, internal IPs, employee names or codes.

## Useful checks

```powershell
curl.exe -s -w " [%{http_code}]" https://champ-hr.com/api/attendance/easytime-import   # 401 = deployed
curl.exe -s -o NUL -w "%{http_code}" --max-time 15 "<easytime>/iclock/api/transactions/?page_size=1"   # 302/401/403 = reachable
curl.exe -s -o NUL -w "%{http_code}" --max-time 15 https://champ-hr.com/login   # 200 = not blocked by VPN
```

`easytime-bridge/dev/mock-easytime.mjs` is a local stand-in EasyTime (`node`, port 8088) for testing
code changes. Never point it at production ESS.

## Report

At the end, write `GO-LIVE-REPORT.md`: each check and result, code changes with diff, open items. No
tokens or passwords.
