# medmetric-tender-notifier

Playwright + eGP tender monitor for medMETRIC. Saves relevant tenders as `PENDING_APPROVAL` for the mobile app.

## Mobile API

| Method | Path | Auth |
|--------|------|------|
| GET | `/api/health` | none |
| GET | `/api/tenders?status=PENDING_APPROVAL` | `X-API-KEY` |
| POST | `/api/tenders/<id>/approve` | `X-API-KEY` |
| POST | `/api/tenders/<id>/reject` | `X-API-KEY` |

Set `MOBILE_API_KEY` in Render (same value in the phone app Settings).

## Environment variables

See [ENV.md](./ENV.md).

## Deploy (Render)

1. Connect this repo  
2. Add Postgres → `DATABASE_URL`  
3. Set env vars from ENV.md  
4. Build: `pip install -r requirements.txt && playwright install chromium`  
5. Start: `gunicorn tender_notifier:app --bind 0.0.0.0:$PORT --workers 1 --threads 4 --timeout 120`

Manual scan: `/test-check?token=$TEST_CHECK_TOKEN`
