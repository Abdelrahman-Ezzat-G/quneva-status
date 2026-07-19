# QuNeva Status Page

[![Status](https://img.shields.io/badge/status-operational-10b981)](https://status.quneva.com)
[![Uptime Check](https://github.com/Abdelrahman-Ezzat-G/quneva-status/actions/workflows/uptime.yml/badge.svg)](https://github.com/Abdelrahman-Ezzat-G/quneva-status/actions/workflows/uptime.yml)

Public status page for [QuNeva WFM](https://www.quneva.com) — Enterprise Workforce Management Platform.

**Live Status Page:** [https://status.quneva.com](https://status.quneva.com)

---

## Monitored Services

| Service | URL | Probe |
|---------|-----|-------|
| QuNeva Website | https://quneva.com | `/healthz` |
| QuNeva AI | https://ai.quneva.com | `/healthz` |

---

## How It Works

- **GitHub Actions** runs uptime checks every ~30 min via an external dispatcher, with a 6-hourly GitHub `schedule` as a fallback heartbeat (`uptime.yml`)
- Results are saved to `data/status.json`, `data/history.csv`, and monthly `data/history-archive-YYYY-MM.csv`
- **GitHub Pages** hosts the status page (`index.html`)
- Incidents are automatically created as GitHub Issues when downtime is detected, and exported to `data/incidents.json` and an Atom feed at `data/incidents.xml`
- Optional email alerts are sent on new incidents and on recovery (see below)
- The status page shows live status, a response-time trend, incidents, and any scheduled maintenance

---

## Email Alerts (optional)

The monitor emails you when a new incident opens and when everything recovers. Alerts fire only on **state changes** (new outage / full recovery), never every hour, and the step is skipped harmlessly if no provider is configured.

**Recommended — [Resend](https://resend.com):** add these repository secrets (**Settings → Secrets and variables → Actions**):

| Secret | Required | Default | Notes |
|--------|----------|---------|-------|
| `RESEND_API_KEY` | ✅ | — | API key from the Resend dashboard |
| `MAIL_FROM` | — | `QuNeva Status <status@abdelrahman-ezzat.com>` | Must be a **verified domain** in Resend |
| `MAIL_TO` | — | `abdelrahman-ezzat@outlook.com` | Where alerts are delivered |

**Fallback — plain SMTP:** if you prefer SMTP instead of Resend, set `MAIL_USERNAME` + `MAIL_PASSWORD` (and optionally `SMTP_HOST` / `SMTP_PORT`, defaulting to `smtp-mail.outlook.com:587`). Resend is tried first when `RESEND_API_KEY` is present, otherwise SMTP is used.

---

## Posting a Scheduled Maintenance Notice

Create a GitHub Issue and add the **`maintenance`** label. While the issue is open it appears as a banner on the status page (title + first lines of the body). Close the issue to remove the banner. Data is written to `data/maintenance.json`.

---

## Repository Structure

```
quneva-status/
├── index.html                    ← Status page (hosted on GitHub Pages)
├── 404.html                      ← Branded not-found page
├── CNAME                         ← Custom domain: status.quneva.com
├── manifest.webmanifest          ← PWA manifest (add to home screen)
├── robots.txt / sitemap.xml      ← SEO
├── og.png / icon-*.png           ← Social share image + PWA/apple icons
├── README.md                     ← This file
├── SETUP_GUIDE.md                ← Developer setup guide
├── data/
│   ├── status.json               ← Latest check results (auto-updated)
│   ├── history.csv               ← Rolling recent history, last 500 rows (auto-updated)
│   ├── history-archive-*.csv     ← Permanent monthly archive (append-only)
│   ├── archives.json             ← Manifest of available archives
│   ├── incidents.json            ← Recent incidents, last 30 days (auto-updated)
│   ├── incidents.xml             ← Atom/RSS feed of incidents (auto-updated)
│   └── maintenance.json          ← Open maintenance notices (auto-updated)
└── .github/
    └── workflows/
        ├── uptime.yml            ← Uptime monitor (~30 min via dispatcher; 6h fallback)
        └── deploy.yml            ← GitHub Pages deployment
```

---

## Adding a New Monitor

Edit `.github/workflows/uptime.yml` and add a new step after the existing checks:

```yaml
- name: Check New Service
  id: newservice
  run: |
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" --max-time 15 https://www.quneva.com/new-page)
    TIME=$(curl -s -o /dev/null -w "%{time_total}" --max-time 15 https://www.quneva.com/new-page)
    echo "status=$STATUS" >> $GITHUB_OUTPUT
    echo "time=$TIME" >> $GITHUB_OUTPUT
    [ "$STATUS" -ge 200 ] && [ "$STATUS" -lt 400 ] && echo "result=up" >> $GITHUB_OUTPUT || echo "result=down" >> $GITHUB_OUTPUT
```

Then add the service to `SERVICES` array in `index.html`.

---

## DNS Setup (Cloudflare)

```
Type:   CNAME
Name:   status
Target: Abdelrahman-Ezzat-G.github.io
Proxy:  DNS Only (grey cloud — NOT orange)
TTL:    Auto
```

---

## GitHub Actions Permissions Required

Go to **Settings → Actions → General**:
- Workflow permissions: **Read and write permissions** ✅
- Allow GitHub Actions to create and approve pull requests ✅

---

*© 2026 QuNeva WFM — Developed by [Abdelrahman Ezzat](https://www.abdelrahman-ezzat.com)*

