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
| `MERKATO_USER` | your 2merkato login | Playwright login |
| `MERKATO_PASS` | password | Playwright login |
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
| `MERKATO_TIME_BUDGET_SEC` | `420` | Hard time limit for the 2merkato part of a scan |
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
POST /api/tenders/<id>/approve
POST /api/tenders/<id>/reject
POST /api/tenders/<id>/drafts   {technical, financial}  -> DOCS_READY (backup of phone drafts)
POST /api/scan                  start a scan now
GET  /api/scan-report           why 2merkato/eGP did or didn't return tenders
```

Read the report: `curl -H "X-API-KEY: $MOBILE_API_KEY" https://YOUR-SERVICE.onrender.com/api/scan-report`

> Render's free Postgres expires after 30 days. If it expires, every tender (including approved ones) is deleted. Upgrade it or export regularly.

Header: `X-API-KEY: <MOBILE_API_KEY>`

## Flow

1. Render Playwright + eGP scrape → AI → save as `PENDING_APPROVAL` in Postgres  
2. App **Pull from cloud** → copies into phone SQLite → you Approve / Reject  
3. App posts Approve/Reject to Render → Postgres updated; Approve can start doc prep  

## Deploy

```bash
# Build
pip install -r requirements.txt && playwright install chromium && playwright install-deps chromium || true
# Start
gunicorn tender_notifier:app --bind 0.0.0.0:$PORT --workers 1 --threads 4 --timeout 120
```

Or use `render.yaml` + linked Postgres database.
