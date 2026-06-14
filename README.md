# 🎬 YT Tag Pilot — YouTube Tag Automation Dashboard

A full-stack dashboard to automate YouTube tag updates across all your managed channels using Google Trends, refreshing every 20 hours.

---

## 🗂 Project Structure

```
yt-dashboard/
├── backend/
│   ├── main.py                  # FastAPI server + scheduler
│   ├── auth_helper.py           # OAuth helper script
│   ├── requirements.txt
│   └── services/
│       ├── youtube_service.py   # YouTube Data API v3
│       └── trends_service.py    # Google Trends (pytrends)
└── frontend/
    ├── index.html
    ├── package.json
    ├── vite.config.js
    └── src/
        ├── main.jsx
        └── App.jsx              # Full dashboard UI
```

---

## ⚡ Quick Start

### Step 1 — Google Cloud Setup (One-time)

1. Go to **https://console.cloud.google.com**
2. Create a new project (e.g. `yt-tag-pilot`)
3. Enable **YouTube Data API v3**:
   - APIs & Services → Library → search "YouTube Data API v3" → Enable
4. Create OAuth 2.0 credentials:
   - APIs & Services → Credentials → Create Credentials → OAuth client ID
   - Application type: **Desktop app**
   - Download the JSON → save as `backend/client_secrets.json`
5. Add your Google email to **Test users** (OAuth consent screen)

---

### Step 2 — Backend Setup

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Authenticate your first account
python auth_helper.py client_secrets.json token_account1.json
# A browser will open → sign in with your Google account
# Copy the printed JSON into the dashboard when connecting
```

Start the backend:
```bash
uvicorn main:app --reload --port 8000
```

Backend runs at: **http://localhost:8000**
API docs: **http://localhost:8000/docs**

---

### Step 3 — Frontend Setup

```bash
cd frontend

npm install
npm run dev
```

Dashboard runs at: **http://localhost:3000**

---

## 🔑 Connecting YouTube Accounts

Each Google account you manage needs a separate OAuth token.

```bash
# For each client account:
python auth_helper.py client_secrets.json token_clientname.json
```

Then in the dashboard:
1. Click **"+ Connect Account"**
2. Go through Google OAuth
3. **All channels on that Google account appear automatically** — including any future ones added to the same email

---

## ⚙️ How the Automation Works

```
Every 20 hours (per account):
│
├── 1. Fetch last 10 videos/shorts via YouTube Data API
├── 2. Detect channel category (Gaming, Tech, Cooking, etc.)
├── 3. Query Google Trends for that category + region
├── 4. Fetch related/rising keywords via pytrends
├── 5. Merge trending keywords + category rules
├── 6. Update each video's tags via YouTube API
└── 7. Log all changes (old tags → new tags, timestamp)
```

---

## 📋 Dashboard Features

| Feature | Description |
|---|---|
| **Account Sidebar** | All connected channels with automation status |
| **Auto-detect** | New channels on connected Google emails appear automatically |
| **Videos Tab** | Last 10 videos/shorts with current tags |
| **Logs Tab** | Full tag change history with diff viewer |
| **Tag Rules** | Category-based base tags that always get included |
| **Trends Tab** | Live trending keywords for the channel's category |
| **Run Now** | Manual trigger per account |
| **20hr Auto** | Enable/disable per-account scheduled automation |
| **Countdown** | Shows time until next automatic run |

---

## 🌍 Supported Regions for Trends

`US`, `IN`, `GB`, `CA`, `AU`, `DE`, `FR`, `BR`, `MX`, `JP`

Change the default in `trends_service.py → get_trending_keywords(region="IN")`

---

## 📦 Tech Stack

| Layer | Tech |
|---|---|
| Backend | Python 3.11+, FastAPI, uvicorn |
| YouTube | google-api-python-client (YouTube Data API v3) |
| Trends | pytrends (Google Trends scraper) |
| Scheduler | asyncio background task loop |
| Frontend | React 18, Vite |
| Auth | Google OAuth 2.0 |

---

## 🔒 Scopes Required

```
https://www.googleapis.com/auth/youtube
https://www.googleapis.com/auth/youtube.force-ssl
```

These allow reading channel data and writing video metadata (tags).

---

## 💡 Production Deployment

For production use, replace the in-memory store with a real database:

```bash
pip install sqlalchemy alembic psycopg2-binary  # PostgreSQL
# or
pip install sqlalchemy aiosqlite               # SQLite (simple)
```

Use a proper task queue for scheduling:
```bash
pip install celery redis
```

Deploy backend with:
```bash
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker
```

---

## ⚠️ YouTube API Quota

YouTube Data API v3 has a **10,000 units/day** free quota.

- Each video tag update = ~50 units
- 10 videos × 50 units = 500 units per account run
- You can run ~20 accounts per day on free quota

For more: Request a quota increase in Google Cloud Console.

---

## 🐛 Troubleshooting

**"Access blocked" on OAuth** → Add email as test user in OAuth consent screen

**"quotaExceeded" error** → You've hit the daily YouTube API limit. Wait 24h or request increase.

**pytrends rate limit** → Google Trends throttles requests. The service has built-in delays.

**Channels not showing** → Make sure you're connecting with the Google account that has manager/editor access on the channels.
