# Marketplace Infrastructure & Deployment Guide (`imshuka.am`)

## 1. Recommended $0-to-Launch Architecture

| Layer | Service | Free Tier Allowance | Purpose & Notes |
| :--- | :--- | :--- | :--- |
| **Frontend & API Hosting** | **Cloudflare Workers / Pages** | Unlimited static bandwidth, 100,000 Workers requests/day | Commercially permitted on the free tier (unlike Vercel Hobby). |
| **Database & Auth** | **Supabase** | 500 MB PostgreSQL (~50,000+ product rows), 50,000 MAUs | Handles user/seller accounts, logins, and product listings. |
| **Image Storage** | **Cloudflare R2** | 10 GB storage/month, **$0 egress (bandwidth) fees** | Stores seller logos and product photos with zero bandwidth charges. |
| **Domain (`.am`)** | **Name.am** | **8,900 AMD/year** (~742 AMD/mo) | Minimum registration is 1 year; supports ArCa & Idram; free WHOIS privacy. |
| **Domain (`.com`)** | **Cloudflare Registrar** | ~$9–$10/year (wholesale) | Zero renewal markup, free WHOIS privacy, instant DNS integration. |
| **Human Email Inbox** | **Cloudflare Email Routing + Gmail** (or **Zoho Mail Free**) | **$0/month** | Forwards `support@imshuka.am` to Gmail, or gives 5 free webmail inboxes on Zoho. |
| **Transactional Email API** | **Resend** (or **Brevo**) | **$0/month** (3,000 emails/mo on Resend; 300/day on Brevo) | Sends automated signup verification, password resets, and order alerts. |

---

## 2. Hosting Comparison & Rules

### Vercel vs. Cloudflare
* **Vercel Free (Hobby):** Strictly **non-commercial** (testing/prototypes only). Live commercial marketplaces require **Vercel Pro ($20/month)**. Free limits: 100 GB bandwidth/mo, 1M function invocations/mo, 4 hrs Active CPU, 5,000 image transformations/mo.
* **Cloudflare Pages / Workers (Free):** **Allows commercial projects for free.**
  * **Static Next.js (`output: 'export'`):** Deploy directly to **Cloudflare Pages** (unlimited bandwidth).
  * **Full-Stack Next.js (SSR / App Router):** Deploy to **Cloudflare Workers** using `@opennextjs/cloudflare` or `vinext`. *Note: Free Workers have a **3 MiB compressed bundle size limit**; larger apps require the **$5/mo Workers Paid plan** (15 MiB limit).*
  * **Express.js Backend:** Deploy to **Cloudflare Workers** using `httpServerHandler` from `cloudflare:node` with `nodejs_compat` enabled (or use **Hono** for a smaller bundle size).

### Deploying Without a Custom Domain
* Hosting providers give a free HTTPS subdomain (e.g., `imshuka.pages.dev` or `imshuka.<subdomain>.workers.dev`).
* **Works identically** for UI, database, auth, and uploads.
* **Limitations:** Lower buyer/seller trust, cannot use custom `@imshuka.am` emails, and Armenian payment gateways (Idram, ArCa, Telcell) generally require a real domain and registered business.

---

## 3. Image Optimization Strategy (Cloudflare R2)

* **Compression Savings (from a 5 MB original photo):**
  * **WebP (Quality 80):** ~**0.75 MB – 1.75 MB** (65%–85% smaller).
  * **AVIF (Quality 65–75):** ~**0.50 MB – 1.25 MB** (75%–90% smaller).
  * **Resized + WebP (1200px–1600px max width):** ~**150 KB – 400 KB** (allows **34,000+ product images** inside R2's 10 GB free tier).
* **Format Rules:**
  * **Product Images:** Convert to **WebP** (Quality 80, max **1200px–1600px** longest side; optional **400px** thumbnails for grid cards) on the client side before uploading to R2.
  * **Seller Logos:** Convert to **WebP** (square **256×256 px** or **512×512 px**, ~10–30 KB). Preserves transparent backgrounds and avoids security risks from user-uploaded SVGs.
  * **Site Hero Banners / UI Icons:** Use **AVIF** (with WebP fallback) for large hero images, and **SVG** for your own UI icons.

---

## 4. Domain & Email Pricing (`imshuka.am`)

### Domain Registration (`Name.am`)
* **1 year:** 8,900 AMD/year (minimum upfront payment; ~742 AMD/month equivalent)
* **2 years:** 8,700 AMD/year (17,400 AMD total)
* **3 years:** 8,500 AMD/year (25,500 AMD total)
* **4 years:** 8,200 AMD/year (32,800 AMD total)
* **5 years:** 7,900 AMD/year (39,500 AMD total)
* **Activation time:** Up to 1 business day after payment.

### Custom Email Options (`@imshuka.am`)
1. **Cloudflare Email Routing + Gmail ($0/mo):** Free forwarding to your personal Gmail + reply via Gmail "Send mail as".
2. **Zoho Mail Free ($0/mo):** Up to 5 mailboxes (5 GB each) via webmail; paid **Mail Lite** starts at **$1/user/mo** for IMAP/POP3.
3. **Google Workspace via Name.am (3,290 AMD/mo / ~$8.89/mo):** Full Gmail, 30 GB Drive, and Meet on your custom domain.
4. **Automated App Emails ($0/mo):** Use **Resend** (3,000 emails/mo free) or **Brevo** (300 emails/day free) via API/SMTP for verification codes and notifications.

---

## 5. When to Upgrade Later
* **Supabase Pro ($25/mo):** When database exceeds 500 MB or daily automated backups are needed.
* **Cloudflare Workers Paid ($5/mo):** If your Next.js/Express Worker bundle exceeds 3 MiB compressed.
* **Railway (~$5–$15/mo):** If you add heavy, long-running background jobs or cron workers.
