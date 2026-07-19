# QuNeva Status Page — Developer Setup Guide

Complete guide to setting up or restoring the QuNeva status page from scratch.

---

## Prerequisites

- GitHub account with access to `Abdelrahman-Ezzat-G/quneva-status`
- Access to Cloudflare DNS panel for `quneva.com`
- No coding required — all steps use GitHub web UI

---

## Step 1: Upload all files to GitHub

If the repository is empty, upload these files via GitHub web UI.

Go to: https://github.com/Abdelrahman-Ezzat-G/quneva-status

For each file, click **Add file → Create new file**, enter the filename and paste the content.

### Files to create:

| Path | Description |
|------|-------------|
| `index.html` | The status page itself |
| `CNAME` | Custom domain (contains: `status.quneva.com`) |
| `README.md` | Repository documentation |
| `SETUP_GUIDE.md` | This file |
| `.github/workflows/uptime.yml` | Uptime monitor workflow |
| `.github/workflows/deploy.yml` | GitHub Pages deploy workflow |

> **Tip:** For files inside `.github/workflows/`, type the full path in the filename field: `.github/workflows/uptime.yml`

---

## Step 2: Set GitHub Actions permissions

Go to: https://github.com/Abdelrahman-Ezzat-G/quneva-status/settings/actions

Under **Workflow permissions**:
- Select: **Read and write permissions** ✅
- Check: **Allow GitHub Actions to create and approve pull requests** ✅

Click **Save**.

This is required for the uptime workflow to commit `data/status.json` back to the repo.

---

## Step 3: Enable GitHub Pages

Go to: https://github.com/Abdelrahman-Ezzat-G/quneva-status/settings/pages

Under **Source**:
- Select: **GitHub Actions**

Click **Save**.

> Note: With `deploy.yml` in place, GitHub Pages is deployed automatically via Actions — not from a branch. Select "GitHub Actions" as the source, not "Deploy from a branch".

---

## Step 4: Set custom domain in GitHub Pages

In the same **Pages** settings page, under **Custom domain**:

Enter: `status.quneva.com`

Click **Save**.

GitHub will attempt to verify DNS. Complete Step 5 first if verification fails.

---

## Step 5: Configure Cloudflare DNS

Log in to Cloudflare: https://dash.cloudflare.com

Select the `quneva.com` zone → **DNS** → **Add record**:

```
Type:   CNAME
Name:   status
Target: Abdelrahman-Ezzat-G.github.io
Proxy:  DNS Only (grey cloud — NOT orange/proxied)
TTL:    Auto
```

Click **Save**.

> **Critical:** The proxy MUST be off (grey cloud). If Cloudflare is proxying the request, GitHub Pages cannot verify domain ownership and HTTPS will not work.

---

## Step 6: Trigger first deployment

Go to: https://github.com/Abdelrahman-Ezzat-G/quneva-status/actions

You should see two workflows:
- **Uptime Monitor** — runs ~every 30 min (external dispatcher) with a 6-hourly fallback
- **Deploy Status Page** — deploys on every push

Click **Uptime Monitor** → **Run workflow** → **Run workflow** (green button).

Wait ~60 seconds, then click **Deploy Status Page** → **Run workflow**.

---

## Step 7: Verify

After DNS propagates (5 min to 24 hours):

1. Open: https://status.quneva.com
2. Should show QuNeva branding with two services
3. Check: https://github.com/Abdelrahman-Ezzat-G/quneva-status/actions — both workflows green ✅
4. Check: `data/status.json` exists in the repo (created by first uptime run)

---

## Troubleshooting

### Status page shows 404
- GitHub Pages may not be enabled — check Settings → Pages
- The deploy workflow may not have run yet — trigger it manually
- DNS may not have propagated — test with `dig status.quneva.com CNAME`

### Actions failing with "Permission denied"
- Check workflow permissions are set to **Read and write** (Step 2)
- If the uptime workflow can't push data: this is expected on first run — re-run after permissions are set

### Custom domain not working / HTTPS errors
- Verify Cloudflare proxy is **OFF** (grey cloud)
- CNAME target must be exactly: `Abdelrahman-Ezzat-G.github.io`
- DNS propagation can take up to 24 hours — be patient

### Uptime checks showing wrong results
- The GitHub Actions workflow (server-side `curl`) is the single authoritative uptime source
- The page renders `data/status.json` (latest check) and `data/history.csv` + archives (history) — it does not probe the services from the browser
- `data/status.json` is updated on every check (~30 min) with accurate results
- If the page looks stale, confirm the latest **Uptime Monitor** run succeeded and committed to `main`

### Workflow runs but data/ files not created
- Check that workflow permissions allow writing (Step 2)
- Check the Actions log for git push errors
- If the repo has branch protection, disable it for the `main` branch

---

## Maintenance

### Check cadence
Primary checks come from an **external dispatcher** (~every 30 min) that triggers
the workflow via `workflow_dispatch`. The GitHub `schedule` in `uptime.yml` is only
a **fallback heartbeat** (`0 */6 * * *`) so the monitor never goes fully dark if the
external trigger stops. To change the fallback frequency, edit that `cron` line:
```yaml
cron: "0 */6 * * *"   # every 6 hours (current fallback)
cron: "0 */2 * * *"   # every 2 hours
cron: "0 * * * *"     # hourly
```
> Note: each run commits to `main`, so shorter intervals mean many more commits.

### Sending a test alert email
Go to **Actions → Uptime Monitor → Run workflow**, tick **test_email**, and run it.
If `RESEND_API_KEY` is set correctly you'll receive a test email at `MAIL_TO`.

### Adding a new monitored service
1. Add a new step in `uptime.yml` (copy an existing check step)
2. Add the service to `SERVICES` array in `index.html`
3. Commit and push

### Viewing uptime history
Raw history is in `data/history.csv` — open it in Excel or Google Sheets.

Format: `timestamp, website_result, website_http, website_time, ai_result, ai_http, ai_time`

---

*© 2026 QuNeva WFM — Maintained by [Abdelrahman Ezzat](https://www.abdelrahman-ezzat.com)*
