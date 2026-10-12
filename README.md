# Link.io — full-stack starter

Responsive social/link-sharing platform scaffold using **Supabase** for accounts, database, and row-level security, with **Paystack** for Nigerian Naira promotion payments.

## Included
- Email/password sign-up and login through Supabase Auth
- Profiles, posts, links, channels, channel memberships, likes, follows, comments
- Row Level Security policies
- Campaign creation and Paystack checkout initialization
- Paystack webhook signature verification; paid campaigns enter `paid_review`
- Mobile-responsive frontend

## Important status
This is a starter, not a finished public production service. It has no admin review dashboard, automated ad-delivery/recommendation engine, email templates, rate limiting, content-reporting UI, or automated tests yet. Campaign payments are **not** a promise of 1,000 followers or a guaranteed audience. Start with manual review before running any promotion.

## 1. Create a Supabase project
1. Create a project at https://supabase.com/ and save the database password securely.
2. In **SQL Editor**, run the entire `supabase/schema.sql` file.
3. Open **Project Settings → API** (or API Keys) and copy the Project URL and publishable/anon key.
4. Edit `index.html`: replace:
   - `YOUR_SUPABASE_PROJECT_URL`
   - `YOUR_SUPABASE_PUBLISHABLE_OR_ANON_KEY`
5. The publishable/anon key is intended for browser use only because the database is protected by RLS. **Never put the service-role key or Paystack secret key in `index.html`.**

## 2. Run the frontend
For quick local testing, a local web server is better than opening the file directly. Install Node.js, then from this folder run:

```bash
npx serve .
```

Open the local URL printed in the terminal. You can also deploy `index.html` to a static host such as Netlify, Vercel, or Cloudflare Pages. Configure the final deployed origin in Supabase Auth → URL Configuration (Site URL and allowed redirect URLs).

## 3. Configure Supabase Auth
In Supabase **Authentication → URL Configuration**, set your deployed site URL. For production, keep email confirmation enabled and configure SMTP so confirmation emails are reliably delivered. Test sign-up, confirmation, login, and sign-out.

## 4. Set up Paystack safely
1. Create/verify your Paystack business account and use **test keys** first.
2. Install Supabase CLI and sign in; link the project:
   ```bash
   supabase login
   supabase link --project-ref YOUR_PROJECT_REF
   ```
3. Set server-side secrets (never in the frontend):
   ```bash
   supabase secrets set PAYSTACK_SECRET_KEY=sk_test_REPLACE_ME
   supabase secrets set SITE_URL=https://your-deployed-domain.example
   ```
   `SUPABASE_URL`, `SUPABASE_ANON_KEY`, and `SUPABASE_SERVICE_ROLE_KEY` are normally supplied to Edge Functions by Supabase. Do not expose the service-role key.
4. Deploy functions:
   ```bash
   supabase functions deploy paystack-initiate
   supabase functions deploy paystack-webhook --no-verify-jwt
   ```
5. In Paystack dashboard, configure a webhook URL:
   `https://YOUR_PROJECT_REF.supabase.co/functions/v1/paystack-webhook`
   Subscribe to `charge.success`. Keep the webhook secret key aligned with the Supabase `PAYSTACK_SECRET_KEY` secret.
6. Use Paystack test mode and test cards from Paystack's official documentation. Do not use real money during initial testing.
7. After verifying the flow, switch to live keys only after the business account and compliance requirements are complete.

The initiation function derives amount, owner and campaign from the database; the browser cannot set the price. The webhook checks its HMAC signature, transaction reference, amount, currency and customer email. Successful payment changes a campaign to `paid_review`; an administrator should approve it before sponsored distribution. Configure `SITE_URL` to the deployed website origin so Paystack returns users to the right place.

## 5. What is still needed before public launch
- Admin dashboard for reviewing campaigns and reports.
- Promotion delivery system that distributes sponsored posts, applies budgets/caps, and records impressions/clicks honestly.
- Automated tests and monitoring for payment/webhook edge cases.
- Abuse prevention: rate limits, URL/domain checks, block/report tools, spam detection, age-appropriate policies, and account recovery.
- Privacy policy, terms, community guidelines, refund/cancellation policy, and support contact.
- Production review of database policies, security, accessibility, and payment compliance.
- Pagination, image/video storage, and a recommendation algorithm if you add TikTok-style media feeds.

## 6. Database notes
- RLS is enabled on every client-exposed table.
- Campaign updates are intentionally blocked for normal users; payment state is server-controlled.
- `record_post_click` is a simple demo counter. Before launch, harden click analytics against bots and abuse, and avoid collecting unnecessary personal data.
- Username collisions during sign-up should be handled in the UI; the generated default username includes a short user-ID suffix.
- For large scale, use pagination and server-side feed queries instead of loading the newest 80 posts at once.

## Troubleshooting
- **"Setup needed"**: check both Supabase values in `index.html`.
- **Email confirmation**: check inbox/spam; configure SMTP for production.
- **RLS error**: make sure `schema.sql` ran successfully and the user is signed in.
- **Checkout doesn't open**: check Edge Function logs, `SITE_URL`, Paystack test secret, deployment and campaign ownership.
- **Webhook 401**: verify Paystack secret configured in Edge Function secrets and webhook signature reaches the function unchanged.
