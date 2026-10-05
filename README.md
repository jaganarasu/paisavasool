# PaisaVasool

An original coupons & deals platform — stores, coupons, deals, categories,
click tracking, a newsletter, shopper accounts, and an admin dashboard.
Built with Next.js 14 (App Router), Prisma, PostgreSQL, NextAuth, and Tailwind.

## Stack

- **Framework:** Next.js 14 (App Router, TypeScript)
- **Database:** PostgreSQL via [Neon](https://neon.tech) (Vercel Postgres uses Neon
  under the hood, so either works — Neon direct is simplest)
- **ORM:** Prisma
- **Auth:** NextAuth (Credentials provider) — one `User` table with a `role`
  field (`ADMIN` / `USER`) drives both the admin login and shopper login
- **Styling:** Tailwind CSS

## What's included

| Feature | Where |
|---|---|
| Public site: home, stores, categories | `src/app/page.tsx`, `src/app/stores/*`, `src/app/categories/*` |
| Click tracking / affiliate redirect | `src/app/go/[id]/route.ts` — logs a `Click` row, then 302s to the store |
| Newsletter signup | `src/components/NewsletterForm.tsx`, `src/app/api/newsletter/route.ts` |
| Shopper signup/login/account | `src/app/signup`, `src/app/login`, `src/app/account` |
| Admin login (separate from shopper login) | `src/app/admin/login` |
| Admin dashboard + CRUD for stores/categories/offers | `src/app/admin/*`, actions in `src/lib/admin-actions.ts` |
| Click-stats page | `src/app/admin/stats` |
| Route protection for `/admin/*` | `src/middleware.ts` |

## 1. Local setup

```bash
npm install
cp .env.example .env
```

Fill in `.env`:
- `DATABASE_URL` / `DIRECT_URL` — from your Neon project (see step 2)
- `NEXTAUTH_SECRET` — generate with `openssl rand -base64 32`
- `NEXTAUTH_URL` — `http://localhost:3000` locally
- `SEED_ADMIN_EMAIL` / `SEED_ADMIN_PASSWORD` — your first admin login

Then:

```bash
npm run db:push   # creates all tables from prisma/schema.prisma
npm run seed      # creates your admin user + sample categories/stores/offers
npm run dev
```

Visit `http://localhost:3000` for the public site and
`http://localhost:3000/admin/login` to sign in with the admin credentials
from your `.env`.

## 2. Create the database (Neon)

1. Go to [neon.tech](https://neon.tech) → New Project → name it `paisavasool`.
2. On the project dashboard, copy the **pooled** connection string into
   `DATABASE_URL` and the **direct** connection string into `DIRECT_URL`
   (Neon labels these clearly on the connection details page).
3. Run `npm run db:push` locally once to create the tables against this database.

You can instead use Vercel's own Postgres (Storage tab → Create Database →
Postgres) — it's Neon-backed, and Vercel will populate the same env var names
for you automatically when you connect it to the project.

## 3. Deploy to Vercel

1. Push this project to a GitHub repo.
2. In Vercel: **New Project** → import the repo.
3. Framework preset: Next.js (auto-detected). Build command and install
   command can stay default — `postinstall` already runs `prisma generate`.
4. Add environment variables in the Vercel project settings (same names as
   `.env.example`):
   - `DATABASE_URL`, `DIRECT_URL`
   - `NEXTAUTH_SECRET`
   - `NEXTAUTH_URL` → your production URL, e.g. `https://paisavasool.vercel.app`
5. Deploy.
6. After the first deploy, run the seed once against production. Easiest way:
   run it locally with your **production** `DATABASE_URL`/`DIRECT_URL` in
   `.env` temporarily:
   ```bash
   npm run seed
   ```
   (Or open Neon's SQL editor and insert an admin user by hand if you'd rather
   not point your local `.env` at prod.)
7. Visit `https://your-domain/admin/login` and sign in.

## 4. Day-to-day admin use

- `/admin` — dashboard with live counts (stores, offers, clicks, subscribers, shoppers)
- `/admin/stores` — add/edit/delete stores
- `/admin/categories` — add/edit/delete categories
- `/admin/offers` — add/edit/delete deals & coupons, toggle Live/Paused, see per-offer click counts
- `/admin/stats` — click totals for the last 7/30 days, ranked by offer

## 5. Notes on how offers work

- A **Deal** shows a "Grab deal" button. A **Coupon** shows a copy-code button
  plus a secondary "Use at store" link. Both route through `/go/[offerId]`,
  which logs a click and then redirects to the `destinationUrl` you set —
  this is where you'd put an affiliate link.
- Toggling an offer to "Paused" hides it from the public site without
  deleting it (and its click history).
- Shoppers can star/save any offer while logged in; saved offers show up on
  `/account`.

## 6. Rebranding

Colors and type live in `tailwind.config.ts` and `src/app/globals.css`
(the `.tag-card` notch shape). Site name/copy is in `src/app/layout.tsx`
(metadata) and `src/components/Header.tsx` / `Footer.tsx`.
