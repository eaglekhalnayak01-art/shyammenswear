# Shyam Men's Wear — Full-Stack Store

Premium men's clothing storefront + private owner dashboard built with **Next.js 16 (App Router)**,
**PostgreSQL** and **Drizzle ORM**.

---

## 1. Local setup

```bash
npm install
cp .env.example .env          # then fill in DATABASE_URL etc.
npx drizzle-kit push          # create the tables
npx tsx src/db/seed-cli.ts    # optional demo catalogue (46 products, 12 orders)
npm run dev
```

Owner login lives at a secret URL:

```
http://localhost:3000/admin/login?key=<ADMIN_ACCESS_KEY>
```

Default demo owner account (change the password immediately):

| Email | Password |
|---|---|
| `admin@shyammenswear.in` | `Admin@12345` |

---

## 2. Environment variables

| Variable | Required | Purpose |
|---|---|---|
| `DATABASE_URL` | ✅ Yes | PostgreSQL connection string |
| `AUTH_SECRET` | ✅ Yes | Signs session cookies — generate with `openssl rand -hex 32` |
| `ADMIN_ACCESS_KEY` | ✅ Yes | Secret segment of the admin login URL |
| `SITE_URL` | No | Used for SEO metadata / sitemap |
| `WHATSAPP_API_TOKEN` | No | Enables new-order WhatsApp alerts |
| `WHATSAPP_PHONE_NUMBER_ID` | No | Enables new-order WhatsApp alerts |

> `.env` is gitignored. Only `.env.example` is committed, so secrets never reach GitHub.

---

## 3. Deploying from GitHub (Vercel)

### Step 1 — Create a hosted PostgreSQL database
Free options that work out of the box:

* **Neon** — neon.tech → copy the *pooled* connection string
* **Supabase** → Settings → Database → Connection string (URI)

The URL must end with `?sslmode=require`.

### Step 2 — Push the schema once
Run this from your machine, pointing at the hosted database:

```bash
DATABASE_URL="postgresql://user:pass@host/db?sslmode=require" npx drizzle-kit push
```

### Step 3 — Import the repo on Vercel
1. vercel.com → **Add New → Project** → import your GitHub repo
2. Framework preset: **Next.js** (auto-detected)
3. Add the environment variables above in **Settings → Environment Variables**
4. Deploy

### Step 4 — Seed the catalogue (optional but recommended)
```bash
DATABASE_URL="postgresql://user:pass@host/db?sslmode=require" npx tsx src/db/seed-cli.ts
```

---

## 4. Why the build used to fail (and how it was fixed)

| Problem | Fix |
|---|---|
| `src/db/index.ts` threw `DATABASE_URL is required` at **import time**, so `next build` crashed | The client is now lazy — it only errors when a query actually runs |
| `app/(store)/[policy]/page.tsx` used `generateStaticParams`, forcing 4 pages to prerender and hit the database during the build | Removed — the route is fully dynamic |
| No `.gitignore`, so `node_modules/` and `.env` were committed | Added a proper `.gitignore` |
| `drizzle.config.json` hard-coded a localhost database | Replaced with `drizzle.config.ts` that reads `DATABASE_URL` |

The project now builds successfully **with no database configured at all**:

```bash
mv .env .env.bak
npm run build      # ✅ succeeds
mv .env.bak .env
```

---

## 5. Scripts

| Command | Description |
|---|---|
| `npm run dev` | Development server |
| `npm run build` | Production build |
| `npm start` | Serve the production build |
| `npm run typecheck` | TypeScript check |
| `npx tsx src/db/seed-cli.ts` | Load the demo catalogue and orders |
| `npx tsx src/db/seed-cli.ts -- --force` | Wipe and reload all demo data |
| `npx drizzle-kit push` | Apply schema changes |

---

## 6. Feature map

**Storefront** — hero, categories, new arrivals, trending tabs, offers, product listing with
filters/sort/pagination, product detail with gallery + zoom, cart, guest checkout, UPI QR payment,
order confirmation, order tracking, customer accounts with OTP login.

**Owner dashboard** (`/admin/*`, hidden behind a secret URL) — overview analytics, orders with a
status workflow, product CRUD with multi-image upload, inventory with low-stock alerts, customers,
homepage/offer editor, store settings and per-product payment rules (COD / online / both).

**Security** — HMAC-signed httpOnly sessions, scrypt password hashing, login rate limiting with
lockout + maths captcha, server-side authorisation on every admin action, server-enforced per-product
payment rules, and new-order notifications.

---

## 7. Going live checklist

- [ ] Change the owner password (Admin → Settings → Owner password)
- [ ] Replace the placeholder UPI QR with your real one
- [ ] Set a strong `AUTH_SECRET`
- [ ] Change `ADMIN_ACCESS_KEY` and bookmark the new login URL
- [ ] Set `SITE_URL` for correct sitemap/SEO URLs
