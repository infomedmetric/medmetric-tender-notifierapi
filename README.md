# medmetric-tender-notifier

Playwright-based 2merkato + eGP tender monitor for medMETRIC. The 2merkato scraper reads `MERKATO_USER` and `MERKATO_PASS` from the service environment (never source code) and saves relevant tenders as `PENDING_APPROVAL` for the mobile app. Configure those two variables in Render to use the authenticated account. `/api/health` reports whether both are configured and the last scan's separate 2merkato/eGP candidate counts; it does not return credential values.

## Mobile API

| Method | Path | Auth |
|--------|------|------|
| GET | `/api/health` | none |
| GET | `/api/tenders?status=PENDING_APPROVAL` | `X-API-KEY` |
| POST | `/api/scan` | `X-API-KEY` |
| POST | `/api/tenders/<id>/approve` | `X-API-KEY` |
| POST | `/api/tenders/<id>/reject` | `X-API-KEY` |

Set `MOBILE_API_KEY` in Render (same value in the phone app Settings).

In the mobile app, open **Settings → Tenders → Server** to enter the Render HTTPS API address and test Postgres connectivity. The app never connects directly to Postgres and must not contain its `DATABASE_URL`. Under **Source scan**, the phone first syncs existing tender workflow records, then may POST `/api/scan` to request an authenticated eGP + 2merkato scan and pull new results. Repeated mobile-triggered scans have a 10-minute default cooldown (`MOBILE_SCAN_COOLDOWN_MINUTES`). The scraper requires an explicit 2merkato `Bidding: Open` status and skips closed/unknown status; eGP filters known expired/non-submittable notices. If `2merkato login` is not configured, add `MERKATO_USER` and `MERKATO_PASS` privately in Render, then review Render logs. Never paste those credentials into the app or source code.

## Environment variables

See [ENV.md](./ENV.md).

## Deploy (Render)

1. Connect this repo  
2. Add Postgres → `DATABASE_URL`  
3. Set env vars from ENV.md  
4. Build: `pip install -r requirements.txt && playwright install chromium`  
5. Start: `gunicorn tender_notifier:app --bind 0.0.0.0:$PORT --workers 1 --threads 4 --timeout 120`

Manual scan: `/test-check?token=$TEST_CHECK_TOKEN`
