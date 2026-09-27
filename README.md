# Dandiya ticketing portal

Ticket sales, signed QR passes, email delivery and gate scanning.
Stack: Next.js on Vercel, Supabase (database), Resend (email), manual UPI and optional Razorpay.

## Pages

| URL | Who | What |
|---|---|---|
| `/` | Public | Choose a pass, pay by UPI or online |
| `/t/<token>` | Ticket holder | Pass with QR code and live status |
| `/admin` | Organiser (password) | Approve UPI payments, resend emails, search, cancel, CSV export |
| `/scan` | Gate volunteers (PIN) | Phone camera scanner with green / amber / red result |

## How fakes are stopped

1. Every pass is a random code plus a signature made with `TICKET_SECRET`. Nobody can create a valid QR without that secret, so edited or made-up QRs show **Fake ticket**.
2. One QR = one person = one entry. The first scan marks it used in a single atomic database step, so two gates scanning the same screenshot at the same moment still let in only one person. The second shows **Already used, 7:42 PM, Gate 2**.
3. Couple and group passes get a separate QR for each person.
4. A UPI UTR can only be attached to one order, so the same payment can't be reused.
5. Cancelling an order voids its passes instantly.

## Setup (about 30 minutes)

1. **Supabase**: create a project (Mumbai region), open SQL Editor, paste and run `supabase/schema.sql`. Copy the project URL and the `service_role` key from Project Settings > API.
2. **Resend**: sign up, add and verify your sending domain (DNS records), create an API key.
3. **Edit `lib/config.js`**: event name, date, venue, contact, pass prices and capacity.
4. **Vercel**: push this folder to a GitHub repo, import it in Vercel, add every variable from `.env.example`, deploy.
5. Set `SITE_URL` to the final domain and redeploy. Generate `TICKET_SECRET` with `openssl rand -base64 48` and never change it after sales start.

Local run: copy `.env.example` to `.env.local`, fill it in, then `npm install` and `npm run dev`.

## UPI flow (works today)

Buyer pays by QR or UPI app, then types the 12-digit UTR. It appears under **Payments to check** in `/admin`. Match the UTR and amount against your bank or UPI business app, then tap **Approve and email passes**.

Use a business/merchant UPI ID if you can. Personal UPI IDs can hit daily limits or warnings in some apps when many people pay.

## Razorpay (when approved)

1. Add `RAZORPAY_KEY_ID` and `RAZORPAY_KEY_SECRET` in Vercel. A **Pay online** button appears automatically, and passes are issued instantly with no manual approval.
2. In Razorpay Dashboard > Webhooks, add `https://your-domain/api/razorpay/webhook`, tick `payment.captured`, set a secret and put it in `RAZORPAY_WEBHOOK_SECRET`. This issues passes even if the buyer closes the tab right after paying.
3. Keep payments on auto-capture.

Remove `UPI_ID` to switch UPI off once Razorpay is live.

## Event day checklist

- A few days before: buy a pass yourself, approve it, scan it twice, and confirm the second scan shows **Already used**. Then cancel that order.
- Give volunteers the `/scan` link and `STAFF_PIN` (use 6+ digits). Each phone sets its gate name once.
- Scanner phones need internet. Carry a pocket Wi-Fi or hotspot as backup.
- Keep one laptop at a help desk logged into `/admin`: search by phone or name, resend passes, or tap **Let in** for someone whose phone died.
- Download the CSV the night before as an offline backup list.

## Email limits

Resend's free plan has a low daily sending cap. For 500 to 2,000 buyers, upgrade for the sales period (check current limits on resend.com). Failed emails show in `/admin` with a **Resend email** button.
