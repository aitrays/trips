# Atlas Trips — Marrakech excursions (EN / FR)

A bilingual (English + French) website for excursions from Marrakech: Agafay desert,
Ourika Valley, Ouzoud waterfalls, Imlil, Essaouira and the medina.

The project ships **two deployment formats**:

| Format | What you get | Where it runs |
| --- | --- | --- |
| **Static site** (`gh-pages/`) | Full website, EN + FR, per-trip pages, WhatsApp booking, optional client-side PayPal, SEO files | **GitHub Pages (free)** |
| **Full-stack app** (`src/`) | Everything above + PostgreSQL content, password-protected admin panel, secure PayPal order creation & capture | Node host (Vercel, Railway, Render…) |

> GitHub Pages only serves static files. It cannot run Next.js API routes, PostgreSQL
> or the admin panel. That is why the fixed content for the free version lives in
> `content/site.json` instead of the database.

---

## 1. Deploy the free version on GitHub Pages

1. Create a new repository on GitHub and push this project to the `main` branch
   (do **not** upload `node_modules`, `.next`, `.env` or `gh-pages/`).
2. In the repository open **Settings → Pages** and set **Source** to
   **GitHub Actions**.
3. Open `content/site.json` and change at least:
   - `site.whatsappNumber` → your number with country code, digits only
     (example `212612345678`).
   - `site.siteUrl` → `https://<your-username>.github.io/<repo-name>` (used for the
     sitemap and canonical URLs).
   - prices in `priceMad` (Moroccan dirham) and `paypalPriceCents` (euro cents,
     `6900` = 69.00 EUR).
4. Commit and push to `main`. The workflow
   `.github/workflows/deploy-github-pages.yml` builds `gh-pages/` and publishes it.
5. Your site is live at `https://<your-username>.github.io/<repo-name>/` and the
   French version at `.../fr/`.

Preview the static build locally at any time:

```bash
node scripts/build-static-site.mjs
npx serve gh-pages      # or: python3 -m http.server -d gh-pages 8080
```

### What you can edit without coding

Everything lives in [`content/site.json`](content/site.json):

- brand name, WhatsApp number, PayPal client ID, site URL,
- English and French hero/SEO/footer texts,
- the trip list: slug, image, price in MAD, PayPal price in €, title, location,
  duration, description and badge in both languages.

Commit the change → GitHub rebuilds the site automatically.

### Payments on GitHub Pages

- **WhatsApp booking is fully functional** and needs no backend.
- **PayPal** can run through the browser SDK if you add a public
  `paypalClientId` (from your PayPal developer dashboard). This is convenient but
  the paid amount is calculated in the browser, so it is less strict than the
  server-verified flow. For a fully verified payment keep the full-stack version.
- `paypalPriceCents` is in EUR because PayPal does not support MAD charges.

### Custom domain

Add your domain in **Settings → Pages → Custom domain**, then create a `CNAME` file
at the repository root containing the domain, and update `site.siteUrl`.

---

## 2. Run the full-stack version (admin + verified PayPal)

```bash
npm install
npx drizzle-kit push        # creates the PostgreSQL tables
npm run dev
```

Environment variables:

| Variable | Purpose |
| --- | --- |
| `DATABASE_URL` | PostgreSQL connection string |
| `ADMIN_PASSWORD` | password for `/admin` |
| `ADMIN_SESSION_SECRET` | signs the admin session cookie |
| `PAYPAL_CLIENT_ID` / `PAYPAL_CLIENT_SECRET` | server-side PayPal (sandbox by default) |
| `PAYPAL_MODE=live` | switch PayPal to production |
| `NEXT_PUBLIC_SITE_URL` | public URL, used for canonicals, sitemap and PayPal return links |

The admin panel at `/admin` lets you edit prices and copy in both languages,
publish/hide trips and read bookings — stored in PostgreSQL.

---

## Project structure

```
content/site.json                 content for the static (GitHub Pages) site
scripts/build-static-site.mjs     static site generator -> gh-pages/
scripts/static-client.js          booking dialog + optional PayPal (browser)
.github/workflows/…               GitHub Pages CI/CD
src/app/[locale]/                 Next.js pages: /en, /fr, trips, confirmation
src/app/admin/                    password-protected dashboard
src/app/api/                      PayPal + admin APIs (full-stack only)
src/lib/i18n.ts                   EN/FR dictionaries and formatters
src/db/                           Drizzle schema and client
```
