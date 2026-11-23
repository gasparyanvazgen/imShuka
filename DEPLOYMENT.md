# `imshuka.am` Deployment & Setup Checklist (`DEPLOYMENT.md`)

## Phase 1: Domain & DNS Setup (Name.am → Cloudflare)
1. **Register Domain on Name.am:** Complete payment for `imshuka.am` (8,900 AMD/year). Wait up to 1 business day for activation.
2. **Add Site to Cloudflare:**
   * Sign up at [://cloudflare.com](https://://cloudflare.com/) (Free plan).
   * Click **Add a site** → enter `imshuka.am` → select the **Free** plan.
   * Copy the **2 Cloudflare nameservers** assigned to you (e.g., `://cloudflare.com` and `://cloudflare.com`).
3. **Update Nameservers in Name.am:**
   * Log into Name.am → **My Domains** → `imshuka.am` → **Edit Nameservers (`Խմբագրել նեյմսերվերները`)**.
   * Replace the default nameservers with your 2 Cloudflare nameservers and save.
   * Wait 15 minutes to 2 hours until Cloudflare shows **Active**.
4. **Enable SSL/TLS:**
   * In Cloudflare → **SSL/TLS** → set encryption mode to **Full (strict)** and enable **Always Use HTTPS**.

---

## Phase 2: Database & Auth Setup (Supabase)
1. **Create Project:** Go to [supabase.com](https://supabase.com) → **New Project** (select a nearby region like `eu-central-1` Frankfurt for lowest latency to Armenia).
2. **Save Credentials:** Copy your `SUPABASE_URL`, `SUPABASE_ANON_KEY`, and `SUPABASE_SERVICE_ROLE_KEY` from **Project Settings → API**.
3. **Configure Auth Redirects:**
   * Go to **Authentication → URL Configuration**.
   * Set **Site URL** to `https://imshuka.am`.
   * Add `https://imshuka.am**` and `http://localhost:3000/**` to **Redirect URLs**.

---

## Phase 3: Product Image Storage (Cloudflare R2)
1. **Create R2 Bucket:**
   * In Cloudflare Dashboard → **R2 Object Storage** → **Create bucket** (e.g., `imshuka-media`).
2. **Connect Custom Media Subdomain (Free CDN):**
   * Open your bucket → **Settings** → **Public access** → **Connect Domain**.
   * Enter `media.imshuka.am` (Cloudflare automatically adds the DNS record and free SSL).
   * Your uploaded WebP images will now load from `https://imshuka.am...` with **$0 egress fees**.
3. **Generate API Keys (for Presigned Uploads):**
   * In R2 Overview → **Manage R2 API Tokens** → **Create API Token**.
   * Grant **Object Read & Write** permissions on `imshuka-media`.
   * Save `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, and `R2_BUCKET_NAME`.

---

## Phase 4: Custom Email Setup (`@imshuka.am`)

### A. Human Inbox (`support@imshuka.am` → Gmail)
1. In Cloudflare → **Email** → **Email Routing** → **Enable Email Routing**.
2. Cloudflare will automatically add the required **MX** and **SPF (TXT)** DNS records—click **Add records and enable**.
3. Add your personal Gmail as a **Destination address** and click the verification link sent to your Gmail.
4. Create a routing rule: `support@imshuka.am` → **Send to an email** → your Gmail address.

### B. Automated App Emails (`noreply@imshuka.am` via Resend)
1. Sign up at [resend.com](https://resend.com) (Free tier: 3,000 emails/mo) → **Domains** → **Add Domain** → enter `send.imshuka.am` *(using a subdomain keeps automated emails cleanly separated from your main domain reputation)*.
2. Add the **DKIM (TXT)** and **SPF (MX/TXT)** records shown by Resend into Cloudflare **DNS → Records** (set proxy status to **DNS only / grey cloud**).
3. Click **Verify DNS Records** in Resend and copy your `RESEND_API_KEY`.

---

## Phase 5: Deploy Your App (Next.js or Express on Cloudflare Workers)

### Option A: Full-Stack Next.js (`@opennextjs/cloudflare`)
Run these commands in your project root:
```bash
npm install @opennextjs/cloudflare@latest
npm install -D wrangler@latest
```
Create `wrangler.jsonc` in your project root:
```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "main": ".open-next/worker.js",
  "name": "imshuka",
  "compatibility_date": "2025-03-01",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "directory": ".open-next/assets",
    "binding": "ASSETS"
  }
}
```
Build and deploy:
```bash
npx opennextjs-cloudflare build
npx wrangler deploy
```

### Option B: Express.js API on Cloudflare Workers
Create `wrangler.jsonc`:
```jsonc
{
  "$schema": "node_modules/wrangler/config-schema.json",
  "name": "imshuka-api",
  "main": "src/index.ts",
  "compatibility_date": "2025-03-01",
  "compatibility_flags": ["nodejs_compat"]
}
```
Wrap your Express `app` using `httpServerHandler` from `cloudflare:node`:
```ts
import express from "express";
import { httpServerHandler } from "cloudflare:node";

const app = express();
app.get("/health", (req, res) => res.json({ ok: true }));

app.listen(8080);
export default httpServerHandler({ port: 8080 });
```








