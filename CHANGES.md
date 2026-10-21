# Tender notifier v2

Based on your uploaded notifier + the earlier fixes (404 on unknown ids, no phantom rows, no fake DOCS_READY stub).

## 2merkato
- Logs whether MERKATO_USER / MERKATO_PASS are set, and verifies the login (login_ok).
- Hard time budget (MERKATO_TIME_BUDGET_SEC, default 420) so it can't hang the scan.
- Fallback crawls `/tenders?page=N`, not just page 1.
- Counters per stage (raw links, title-relevant, closed, already notified, candidates) -> `GET /api/scan-report`.
- I could not reach tender.2merkato.com from my sandbox: selectors are UNVERIFIED. Run a scan, read `/api/scan-report`
  and the Render logs; they show which stage returns zero.

## Approved tenders
- A re-scan can no longer revert APPROVED/REJECTED/DOCS_* back to PENDING_APPROVAL.
- Drafts are backed up in Postgres (`POST /api/tenders/<id>/drafts`) and returned by `GET /api/tenders`.
- DB helpers now commit/rollback, so one failed query no longer poisons the shared connection.
- Render free Postgres is deleted after 30 days — the most likely way "everything disappears".

## Other
- `POST /api/scan` + `GET /api/scan-report` (scan lock, no overlapping scans).
- HARD_EXCLUDE_TERMS had a missing comma ("road" "laboratory" were fused). Now "road construction"; "laboratory" is still NOT excluded (same effective behaviour as before).
