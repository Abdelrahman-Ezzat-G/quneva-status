# QuNeva Status Page

[![Upptime](https://img.shields.io/endpoint?url=https%3A%2F%2Fraw.githubusercontent.com%2FAbdelrahman-Ezzat-G%2Fquneva-status%2FHEAD%2Fapi%2Fquneva-website%2Fuptime.json)](https://status.quneva.com)

Public status page for [QuNeva WFM](https://www.quneva.com) — Enterprise Workforce Management Platform.

**Status Page:** https://status.quneva.com

## Monitored Services

| Service            | URL                          |
| ------------------ | ---------------------------- |
| QuNeva Website     | https://www.quneva.com       |
| QuNeva Root Domain | https://quneva.com           |
| QuNeva Login       | https://www.quneva.com/login |

## How It Works

This repository uses [Upptime](https://upptime.js.org/) to automatically monitor the QuNeva platform every 5 minutes using GitHub Actions.

- **Uptime checks**: GitHub Actions pings each URL and records response
- **Incident management**: Issues are created automatically when downtime is detected
- **Status page**: Hosted on GitHub Pages at status.quneva.com
- **History**: All uptime history is committed to this repository

## Maintenance

### Adding a new monitor

Edit `.upptimerc.yml` and add to the `sites` list:

```yaml
- name: New Service Name
  url: https://www.quneva.com/new-page
```
