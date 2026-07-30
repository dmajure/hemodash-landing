# HemoDash Landing Page — Deployment Guide

## What this repo is

Two files, no build step:

- `index.html` — the whole page, self-contained (inline CSS + one inline script).
- `icon.svg` — the favicon, copied verbatim from the app's `web/public/icon.svg`.

The design tokens, typefaces, wordmark, and PV-loop animation are **hand-ported
from the HemoDash app** so the two surfaces read as one product. The `:root` block
in `index.html` names its source (`web/src/app/globals.css`), and the PV-loop path
and marker keyframes are copied verbatim from `web/src/lib/loadingLoop.ts`. There
is no automatic sync — re-port by hand if the app's tokens change.

## Email capture

The waitlist form already posts to a live Formspree endpoint
(`https://formspree.io/f/mqewajzd`). Nothing to configure. Submissions land in
that Formspree inbox; the free tier allows 50/month.

Success and failure both render inline in the page — no `alert()`, no redirect.

**Alternative:** to switch to Mailchimp, ConvertKit, or similar, replace the
`<form>` block and delete the `<script>` at the bottom of `index.html`.

---

## Option A: Deploy to Cloudflare Pages (Recommended)

Since your domains are on Cloudflare, this is the fastest path — free hosting, automatic SSL, global CDN.

### Step 1: Push to GitHub
```bash
cd landing-page
git init
git add .
git commit -m "HemoDash coming soon page"
git remote add origin https://github.com/YOUR_USERNAME/hemodash-landing.git
git push -u origin main
```

### Step 2: Connect to Cloudflare Pages
1. Log into your **Cloudflare dashboard** at dash.cloudflare.com.
2. Go to **Workers & Pages** in the left sidebar.
3. Click **Create** > **Pages** > **Connect to Git**.
4. Authorize GitHub and select your `hemodash-landing` repo.
5. Configure the build:
   - **Build command:** leave blank (it's a static HTML file)
   - **Build output directory:** `/` (or `.` — the root of the repo)
6. Click **Save and Deploy**.

Cloudflare will assign you a URL like `hemodash-landing.pages.dev` within ~60 seconds.

### Step 3: Connect Your Custom Domain
1. In Cloudflare Pages, go to your project > **Custom domains**.
2. Click **Set up a custom domain** and enter `hemodash.com`.
3. Cloudflare will automatically configure the DNS record (since the domain is already on Cloudflare).
4. SSL is automatic — your site will be live at `https://hemodash.com` within minutes.

### Step 4: Redirect hemodynamics.app to hemodash.com
1. In your Cloudflare dashboard, go to the **hemodynamics.app** domain.
2. Go to **Rules** > **Redirect Rules**.
3. Create a rule:
   - **When:** Hostname equals `hemodynamics.app`
   - **Then:** Redirect to `https://hemodash.com` (301 permanent redirect)
4. Save and deploy.

Now anyone visiting hemodynamics.app lands on your HemoDash page.

---

## Option B: Deploy to Vercel (Alternative)

### Step 1: Install Vercel CLI
```bash
npm install -g vercel
```

### Step 2: Deploy
```bash
cd landing-page
vercel
```
Follow the prompts. Vercel will give you a live URL instantly.

### Step 3: Add Custom Domain
1. Go to your project on vercel.com > **Settings** > **Domains**.
2. Add `hemodash.com`.
3. Vercel will show you DNS records to add in Cloudflare:
   - Type: `CNAME`, Name: `@`, Target: `cname.vercel-dns.com`
4. Add this record in Cloudflare DNS settings (set proxy status to "DNS only").

---

## Option C: Deploy to Netlify (Alternative)

1. Go to **app.netlify.com** > **Add new site** > **Deploy manually**.
2. Drag and drop the `landing-page` folder.
3. Site goes live instantly at a `.netlify.app` URL.
4. Go to **Domain management** > **Add custom domain** > enter `hemodash.com`.
5. Add the DNS records Netlify provides to your Cloudflare DNS settings.

---

## Post-Deploy Checklist

- [ ] Verify https://hemodash.com loads correctly
- [ ] Test email signup — submit a test email and confirm it arrives in Formspree
- [ ] Test on mobile (iPhone and Android)
- [ ] Verify hemodynamics.app redirects to hemodash.com
- [ ] Submit to Google Search Console (search.google.com/search-console) so Google indexes your page
- [x] Favicon — `icon.svg`, the app's own mark
- [ ] Add an `og:image` and switch `twitter:card` back to `summary_large_image`
      (currently `summary`, because there is no image asset)

## Before launch

- [x] **Intended-use / not-a-medical-device statement** — restored under the
      waitlist note as `.disclaimer-note` (`DISCLAIMER.short` verbatim from
      `web/src/lib/legal.ts`). It is a SIBLING of `#formNote`, not a child: the
      success handler hides the note, and this statement must survive a signup.
      The audience clause lives in the waitlist note above it.
- [ ] **Restore the copyright** (`COPYRIGHT` in `web/src/lib/site.ts`).

## Deliberately absent

- **No links to /about, /methodology, /terms, /privacy, /contact.** That language is
  pending attorney review (`LEGAL_DRAFT` is still `true` in the app), and the app
  itself is not deployed, so those paths would 404. Wire them in the same change
  that points this domain at the deployed app.
- **No social links.** Add when the LinkedIn and Bluesky profiles are confirmed live.
- **No "open the app" CTA.** The page is pre-launch; the waitlist is the only action.
