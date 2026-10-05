# GOOT Resorts — GitHub + Render Demo

This repository is a deployment-ready copy of the GOOT Resorts project.

## Deploy
1. Upload this complete folder to a new GitHub repository.
2. In Render, create a Blueprint from that repository.
3. Render will create the web service and PostgreSQL database from `render.yaml`.
4. Set `ADMIN_KEY` in Render to a new long random demo-only secret.
5. Wait for `/api/health` to return `{"ok":true,"db":true}`.
6. Open `/preview.html` for the buyer-facing demo landing page.
7. Open `/` for the resort website and `/admin.html` for the protected admin dashboard.

## Important
- Do not commit `server/.env`, PostgreSQL data folders, or real production secrets.
- The included database initializer creates the schema automatically.
- The first empty demo database receives three sample bookings so the dashboard is not empty.
- `SEED_DEMO_DATA=false` disables the sample-data seed.
