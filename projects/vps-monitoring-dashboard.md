# VPS monitoring dashboard

**Status:** idea
**Started:** 2026-06-30
**Tags:** #vps #infra

## What is it?
Small dashboard for vm666 — uptime, disk space, basic service health (nginx, FastAPI app, MySQL).

## Why
Now that the site is on a self-managed VPS, there's no shared-hosting safety net. Want visibility before something breaks.

## Brainstorming
- Could be a tiny FastAPI page reusing the app already planned for the VPS
- Cron script (already on the to-do list) could feed it data
- Alerting via email if disk usage crosses a threshold?

## Specs / requirements
- 

## Open questions
- Build vs. just use an existing lightweight tool (e.g. Netdata, Uptime Kuma)?

## Links
- 
