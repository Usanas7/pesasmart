# PesaSmart

A **USSD-based group governance and transparency platform** for informal **Ikimina** (rotating savings) circles in urban Rwanda.

Group **members** interact entirely over **USSD** from any mobile phone — no smartphone or internet required — while group **organisers** manage their groups through a **web admin panel**. This keeps the platform inclusive of every member regardless of device, while leaving the group's money flow peer-to-peer (PesaSmart is **non-custodial** — it never holds funds).

## Repositories

This project is split into two repositories:

- **Frontend — organiser admin panel:** https://github.com/Usanas7/pesasmart-frontend
- **Backend — API, USSD & database:** https://github.com/Usanas7/pesasmart-backend

## Live Links

- **Web app:** https://pesasmart.vercel.app
- **API:** https://pesasmart-backend.onrender.com
- **Demo video:** https://youtu.be/WiymB89qsZs

## How It Works

- **Members** dial a USSD code (via the Africa's Talking gateway) to view rotation status, raise contribution disputes, and request membership changes — all on a basic phone.
- **Organisers** sign in to the web admin panel to manage groups, confirm contributions, and resolve disputes.
- **Group-wide accountability notifications** are pushed to members via SMS.

## Tech Stack

- **Frontend:** React (Vite) + React Router — hosted on Vercel
- **Backend:** Node.js + Express — hosted on Render
- **Database:** PostgreSQL — hosted on Neon
- **USSD / SMS:** Africa's Talking
- **CI/CD:** GitHub Actions (CI) + Render/Vercel auto-deploy (CD)

## Deployment Plan

| Component | Host | Notes |
|---|---|---|
| Frontend | Vercel | Auto-deploys on push to `main` |
| Backend | Render | Auto-deploys on push to `main`; `DATABASE_URL` as an environment variable |
| Database | Neon | Managed cloud PostgreSQL |
| USSD | Africa's Talking (sandbox) | Callback set to the backend `/ussd` route |
| CI | GitHub Actions | Runs on every push in each repo |

## Setup

Each repository has full setup instructions in its own README. In short:

- **Backend:** clone `pesasmart-backend`, `npm install`, add a `.env` with `DATABASE_URL`, then `npm start`.
- **Frontend:** clone `pesasmart-frontend`, `npm install`, add a `.env` with `VITE_API_URL`, then `npm run dev`.
