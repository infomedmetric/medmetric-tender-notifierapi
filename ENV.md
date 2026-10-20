# Render environment variables — medMETRIC Tender Agent

Set these in **Render Dashboard → your Web Service → Environment**.

## Required

| Variable | Example | Purpose |
|----------|---------|---------|
| `DATABASE_URL` | *(from Render Postgres)* | Stores tenders + statuses |
| `GROQ_API_KEY` | `gsk_...` | AI relevance scoring |
| `MOBILE_API_KEY` | long random string | App uses this as `X-API-KEY` |
| `TEST_CHECK_TOKEN` | random string | Manual scan: `/test-check?token=...` |

## Recommended

| Variable | Example | Purpose |
|----------|---------|---------|
| `GROQ_MODEL` | `llama-3.1-8b-instant` | Groq model id |
| `GROQ_API_KEY_EGP` | second `gsk_...` | Extra quota for eGP batch (optional) |
| `SCAN_INTERVAL_HOURS` | `6` | Background scan interval |
| `MOBILE_SCAN_COOLDOWN_MINUTES` | `10` | Minimum interval between mobile-triggered source scans |
| `MERKATO_USER` | your 2merkato login | Authenticated Playwright login; set privately in Render, never in source code or chat |
| `MERKATO_PASS` | password | Authenticated Playwright login; set privately in Render, never in source code or chat |
| `EGP_USER` | eGP portal user | Optional portal login |
| `EGP_PASS` | eGP password | Optional portal login |
| `EGP_BASE` | `https://production.egp.gov.et` | eGP host |

## Optional WhatsApp (short notify only)

| Variable | Purpose |
|----------|---------|
| `EVOLUTION_BASE` | Evolution API base URL |
| `INSTANCE_NAME` | Instance name |
| `GLOBAL_API_KEY` | Evolution / fallback API key |
| `WHATSAPP_NUMBERS` | Comma-separated numbers |

## Optional tuning

| Variable | Default | Purpose |
|----------|---------|---------|
| `GROQ_TEMPERATURE` | `0.2` | AI temperature |
| `GROQ_MAX_TOKENS` | `2048` | Max completion tokens |
| `AI_CHUNK_SIZE` | `3` | Batch size for AI |
| `MERKATO_MAX_PAGES` | `4` | 2merkato pages |
| `MERKATO_MAX_PAGES_PER_TERM` | `3` | Pages per search term |

## Mobile app settings (not Render)

On the phone **Settings**:

| Field | Example |
|-------|---------|
| Tender API URL | `https://YOUR-SERVICE.onrender.com` |
| Tender API key | same as `MOBILE_API_KEY` |

## API used by the app

```
GET  /api/health
GET  /api/tenders?status=PENDING_APPROVAL
POST /api/scan
POST /api/tenders/<id>/approve
POST /api/tenders/<id>/reject
```

Header: `X-API-KEY: <MOBILE_API_KEY>`

## Flow

1. App syncs existing tender workflow records from Postgres through the HTTPS API (the phone never receives a database connection string).  
2. App can request `POST /api/scan`; the API requires `X-API-KEY`, starts at most one scan at a time, and applies `MOBILE_SCAN_COOLDOWN_MINUTES` (default 10) to protect source/AI quotas. It scans eGP + 2merkato for new currently open notices. IDs already present in Postgres or the notified-ID table are skipped before AI review. New candidates are saved in Postgres as `PENDING_APPROVAL`. Known closed/expired listings are excluded by source filters.  
3. App fetches the active rows into phone SQLite → you Approve / Reject  
4. App posts Approve/Reject to Render → Postgres updated; approved and document-draft statuses can be restored during app pulls.  

`GET /api/health` reports whether the server can connect to Postgres, whether both 2merkato login variables are configured, scan state, and the latest 2merkato/eGP candidate counts. It never returns credentials or the database URL. If counts are zero or null, inspect Render logs for the 2merkato engine's login/page/search errors; configured username/password alone does not prove that the current site selectors or account login are working.

## Deploy

```bash
# Build
pip install -r requirements.txt && playwright install chromium && playwright install-deps chromium || true
# Start
gunicorn tender_notifier:app --bind 0.0.0.0:$PORT --workers 1 --threads 4 --timeout 120
```

Or use `render.yaml` + linked Postgres database.
