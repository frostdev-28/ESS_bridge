# Changes made on the VPN machine (port these to PR #1, `integrations/easytime-bridge/`)

The exact line-by-line diff against the PR #1 version is in `CHANGES.diff`. To apply it in the ESS repo
(on branch `easytime-chrome-bridge`, from the repo root):

```powershell
git apply --directory=integrations CHANGES.diff
```

## 1. Unknown employee codes persist until matched (2026-10-06)

**Problem:** `status.unmatchedCodes` was overwritten by every run that posted punches, so the codes behind
the first catch-up batch (207 unmatched punches) were lost from the popup five minutes later.

**Change:**
- `sync.js`: new `rememberUnmatched(chunk, unmatchedEmpCodes)`, called after every accepted chunk in
  `postAll`. It keeps `chrome.storage.local.unmatched` as `{ [CODE]: { code, from, to } }` (IST punch
  dates). A code is removed when a later chunk contains it and ESS does not report it as unmatched. Codes
  are compared trimmed and upper-cased. If ESS returns a code in a form that matches no record in the chunk,
  the chunk's date range is used.
- `sync.js`: `doSync` no longer writes `unmatchedCodes` into `status`. New exported `clearUnmatched()`,
  which waits for an in-flight sync first.
- `background.js`: new message `clear-unmatched`.
- `popup.html` / `popup.js` / `styles.css`: the list shows a count, one code per line with its dates, a
  **Re-send <from → to>** button that queues a re-send of the whole range, and **Clear list**. The list
  scrolls after 180px.

## 2. Install path in the README (2026-10-06)

`README.md` *Install* step 1 suggested a path under one user's profile. It now suggests
`C:\ESS\easytime-bridge`, which is the same on every machine.

## Testing

Change 1 was **not tested against the mock** (no node on the VPN machine); verified live against real EasyTime/ESS.
