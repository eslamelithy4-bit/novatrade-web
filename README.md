# AlphaNex AI Trade — Backend

Production-grade Node.js + TypeScript backend for the AlphaNex AI Trade investment platform.

## Stack

- Express.js + TypeScript (ESM)
- Drizzle ORM + PostgreSQL
- Redis (cache, sessions, rate limit, BullMQ)
- BullMQ (background jobs: profit/bonus distribution, price fetcher, email worker, cleanup)
- Socket.IO (real-time price updates, notifications)
- JWT auth (access 15m + refresh 30d, httpOnly cookie)
- bcrypt password hashing (12 rounds)
- AWS S3 (or R2 / Supabase Storage compatible)
- Nodemailer (transactional email)
- Zod validation
- Pino structured logging
- Helmet, CORS, rate-limit, slow-down

## Setup

```bash
cd backend
cp .env.example .env       # then edit values
npm install                # or pnpm / bun
npm run db:generate        # generate migration files from schema
npm run db:migrate         # apply migrations
npm run db:seed            # seed default admin, settings, networks
npm run dev                # start dev server with watch
```

## Production

```bash
npm run build
npm start
```

## Environment

All secrets default to placeholder values (`CHANGE_ME`, `YOUR_*_HERE`).
Replace them in `.env` before deploying.

## API Surface

| Group | Base path |
|---|---|
| Auth | `/api/auth` (signup, login, refresh, logout, forgot/reset password, 2FA, me) |
| Users | `/api/users` (me, profile, password, 2FA, KYC, referrals, notifications, admin CRUD) |
| Transactions | `/api/transactions` (deposit, withdraw, claim-bonus, claim-profit, admin approve/reject, CSV export) |
| Settings | `/api/settings` (public, admin bulk update) |
| Networks | `/api/networks` (list, admin CRUD) |
| Content | `/api/content` (articles, faqs, legal) |
| Prices | `/api/prices` (current, history) |
| Admin | `/api/admin` (dashboard, activity logs, KYC review, roles, signup-fields, bonus-tiers, ai-algos, languages, payment-providers) |
| Upload | `/api/upload` (avatar, deposit-proof, kyc, asset) |
| Notifications | `/api/notifications/broadcast` |

Response format:
```json
{ "success": true, "data": ... }
{ "success": false, "error": "...", "details": ... }
```

## Real-Time Events

Connect Socket.IO with `auth: { token: <accessToken> }`.

| Event | Room | Payload |
|---|---|---|
| `price:update` | `prices` | `{ BTCUSDT: 64000, ETHUSDT: 3400, ... }` |
| `notification:new` | `user:{id}` | notification row |
| `transaction:updated` | `user:{id}` | `{ txId, status }` |
| `admin:notification` | `admin` | `{ type, txId, userId, amount }` |

## Background Jobs (BullMQ)

| Job | Schedule | Purpose |
|---|---|---|
| `profit-distributor` | daily 00:05 | Distribute random 1-8% profit to all funded users |
| `bonus-distributor` | daily 06:00 | Daily bonus to eligible users |
| `price-fetcher` | every 5s | Pull prices from Binance, cache, broadcast |
| `email-sender` | on-demand | Send transactional emails |
| `cleanup` | daily 03:00 | Purge expired refresh tokens, archive old logs |

## Notes

- All money is `numeric(18,8)` — never float
- All balance mutations occur in DB transactions with `SELECT FOR UPDATE`
- Soft delete only for users — financial history preserved
- Private KYC files in S3 served via 1h presigned URLs
- First registered user becomes admin automatically
- Referral commissions: L1=10% / L2=3% / L3=1% (configurable via settings)
