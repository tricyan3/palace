# Deploying PALACE Website

This guide explains how to deploy the PALACE website with GitHub Pages and Cloudflare Workers.

## Architecture

```
GitHub Pages (Static Site)     Cloudflare Workers (API)
├── index.html                 ├── /api/checkout
├── download.html              ├── /api/verify  
├── css/                       └── /api/webhook
└── images/                         ↓
        ↓                      Stripe API
   User Browser  ─────────────────────→
```

---

## 1. Stripe Setup

### Create Products in Stripe Dashboard

1. Go to [Stripe Products](https://dashboard.stripe.com/products)
2. Create **PALACE Monthly**:
   - Price: $2.00 USD
   - Billing: Recurring (Monthly)
   - Copy the `price_xxx` ID

3. Create **PALACE Lifetime**:
   - Price: $12.00 USD
   - Billing: One-time
   - Copy the `price_xxx` ID

### Get API Keys

1. Go to [Stripe API Keys](https://dashboard.stripe.com/apikeys)
2. Copy your **Secret key** (`sk_live_xxx` or `sk_test_xxx`)

---

## 2. Cloudflare Workers Deployment

### Install Wrangler CLI

```bash
npm install -g wrangler
wrangler login
```

### Deploy the Worker

```bash
cd workers
npm install
wrangler deploy
```

### Set Environment Variables

In Cloudflare Dashboard (Workers → palace-api → Settings → Variables):

| Variable | Value |
|----------|-------|
| `STRIPE_SECRET_KEY` | `sk_live_xxxxx` (from Stripe) |
| `STRIPE_PRICE_MONTHLY` | `price_xxxxx` (monthly plan) |
| `STRIPE_PRICE_LIFETIME` | `price_xxxxx` (lifetime plan) |

**Important**: Mark `STRIPE_SECRET_KEY` as **Encrypted**.

### Get Your Worker URL

After deploying, you'll get a URL like:
```
https://palace-api.YOUR_SUBDOMAIN.workers.dev
```

---

## 3. Update Website

Edit `web/index.html` and update the API URL:

```javascript
const API_BASE_URL = 'https://palace-api.YOUR_SUBDOMAIN.workers.dev';
```

---

## 4. GitHub Pages Deployment

### Option A: Manual Upload

1. Create a new repository for your website (e.g., `palace-website`)
2. Copy the contents of `/web` folder to the repository
3. Go to Settings → Pages → Enable GitHub Pages from main branch

### Option B: GitHub Actions (Recommended)

1. In your private repo, enable GitHub Pages via Actions
2. Copy `.github/workflows/deploy-pages.yml` to your repo
3. Push to main branch

Your site will be at: `https://YOUR_USERNAME.github.io/REPOSITORY_NAME/`

---

## 5. Set Stripe Webhook

1. Go to [Stripe Webhooks](https://dashboard.stripe.com/webhooks)
2. Add endpoint: `https://palace-api.YOUR_SUBDOMAIN.workers.dev/api/webhook`
3. Select events:
   - `checkout.session.completed`
   - `customer.subscription.created`
   - `customer.subscription.deleted`

---

## Testing

### Test Stripe Checkout
1. Open your website
2. Click "Buy Now"
3. Use Stripe test card: `4242 4242 4242 4242`

### Test Verification
1. Complete a test purchase
2. Use the email verification form

---

## Troubleshooting

### CORS Errors
Make sure your worker has correct CORS headers (already configured in `index.js`).

### Checkout Not Working
1. Check browser console for errors
2. Verify `API_BASE_URL` is correct in `index.html`
3. Check Cloudflare Worker logs: `wrangler tail`

### Price ID Errors
Make sure you've set the environment variables in Cloudflare Dashboard.
