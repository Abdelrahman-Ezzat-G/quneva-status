# QuNeva Status Page

[![Status](https://img.shields.io/badge/status-operational-10b981)](https://status.quneva.com)
[![Uptime Check](https://github.com/Abdelrahman-Ezzat-G/quneva-status/actions/workflows/uptime.yml/badge.svg)](https://github.com/Abdelrahman-Ezzat-G/quneva-status/actions/workflows/uptime.yml)

Public status page for [QuNeva WFM](https://www.quneva.com) — Enterprise Workforce Management Platform.

**Live Status Page:** [https://status.quneva.com](https://status.quneva.com)

---

## Monitored Services

| Service | URL |
|---------|-----|
| QuNeva Website | https://www.quneva.com |
| QuNeva Root Domain | https://quneva.com |
| QuNeva Login | https://www.quneva.com/login |

---

## How It Works

- **GitHub Actions** runs uptime checks every 5 minutes (`uptime.yml`)
- Results are saved to `data/status.json` and `data/history.csv`
- **GitHub Pages** hosts the status page (`index.html`)
- Incidents are automatically created as GitHub Issues when downtime is detected
- The status page fetches live results directly from the monitored URLs

---

## Repository Structure

```
quneva-status/
├── index.html                    ← Status page (hosted on GitHub Pages)
├── CNAME                         ← Custom domain: status.quneva.com
├── README.md                     ← This file
├── SETUP_GUIDE.md                ← Developer setup guide
├── data/
│   ├── status.json               ← Latest check results (auto-updated)
│   └── history.csv               ← Full history log (auto-updated)
└── .github/
    └── workflows/
        ├── uptime.yml            ← Uptime monitor (every 5 min)
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

*© 2026 QuNeva WFM — Developed by [Abdelrahman Ezzat](https://www.abdelrahman-ezzat.vip)*
