# Monox AI — RV Solar & Repair Funnel

A **single-page** lead-gen funnel, static HTML for Vercel. No build step, no frameworks.

```
/               index.html    The whole funnel
/logo.png       monox.ai wordmark (transparent)
/favicon.png    Monox "O" mark (transparent)
```

**Flow (all on one page):** Paid Meta traffic lands on `/` → clicks "See How It Works" →
fills the modal form → the how-it-works **Loom unlocks inline** and the **booking calendar
is revealed** on the same page. No redirect.

---

## What's wired

- **Meta Pixel** `1514677843570094` (hardcoded).
- **GHL Inbound Webhook** — set in `index.html` `CONFIG.WEBHOOK_URL`. ✅
- **How-it-works video** (gated): Loom `610a1353…` — unlocks inline on submit.
- **Case-study video** (free): Chris / Tucson RV Solar, Loom `2dd73e63…`.
- **Logo + favicon** isolated to transparent PNGs. Fonts: Poppins + Inter.

## Fully configured ✅

- **GHL Inbound Webhook** → `CONFIG.WEBHOOK_URL`.
- **GHL Booking widget** → `CONFIG.BOOKING_URL` (reveals inline on unlock).
- **Blurred how-it-works teaser** → `howitworks-thumb.jpg` (a real frame of the Loom).
- **No `{{placeholders}}` remain.**

---

## Conversion tracking (IMPORTANT)

The **Lead** event = your conversion. It fires **only when a prospect's contact info is
captured and confirmed received by the CRM** — never on a plain page view.

Flow on submit:
1. The form unlocks the video immediately (the prospect always gets access).
2. In the background it POSTs the contact to `CONFIG.WEBHOOK_URL`.
3. **Only if the CRM responds OK**, it fires `Lead` once — with advanced matching
   (em/ph/fn from the form) and an `eventID` for CAPI dedup.

So:
- `PageView` (load) and `ViewContent` (form open) fire as traffic/retargeting signals —
  **not** conversions. **Optimize your ad set on `Lead`.**
- If the webhook ever fails, the user still gets the video but **no Lead is counted.**

---

## Map the form into GoHighLevel (name, email, phone)

The form POSTs this JSON to the Inbound Webhook:

```json
{
  "name": "Jane Doe", "first_name": "Jane", "last_name": "Doe",
  "email": "jane@example.com", "phone": "+15551234567",
  "looking_to_grow": "Yes", "source": "RV Solar Funnel", "page": "rv-solar-funnel",
  "submitted_at": "2026-01-01T00:00:00.000Z",
  "utm_source": "...", "utm_campaign": "...", "fbclid": "...", "fbc": "...", "fbp": "..."
}
```

In the workflow (trigger = Inbound Webhook):
1. **Fetch Sample Request**, then submit one real test on `/` so GHL sees the fields.
2. **Create/Update Contact** → First Name ← `{{inboundWebhookRequest.first_name}}`,
   Last Name ← `{{inboundWebhookRequest.last_name}}`, Email ← `{{inboundWebhookRequest.email}}`,
   Phone ← `{{inboundWebhookRequest.phone}}`. (Full name is also sent as `name`.)
3. **Update Contact Field** → `Looking To Grow` ← `{{inboundWebhookRequest.looking_to_grow}}` (create the field, or use it to branch / add a tag).
4. Add Tag `rv-solar-funnel-lead` + Internal Notification to Luke + Isaac.

> GHL's inbound webhook returns `200` and permits CORS, so the page reads success and
> fires the Lead conversion. Dedupe contacts by email + phone.

## Booking automations (separate workflow)

Trigger **"Customer Booked Appointment"** on this calendar → confirmation email + SMS →
reminders 24h + 1h before → no-show SMS + rebooking link / showed → post-call stage.

---

## Deploy

- Repo `github.com/stukienukie/rv-solar-funnel`; Vercel project **rv-solar-funnel**.
- Production branch **`main`** auto-deploys on push → https://rv-solar-funnel.vercel.app/
- Custom domain recommended (e.g. `go.monox.ai`).

## Test checklist

- [ ] Submit on `/` → video unlocks inline, booking reveals (once URL set), contact lands
      in GHL with tag + `Looking To Grow` + UTM/fbclid fields.
- [ ] Meta **Test Events**: `PageView`, `ViewContent` (form open), and a single `Lead`
      with advanced matching — **only** after a successful CRM capture.
- [ ] Break `CONFIG.WEBHOOK_URL` → video still unlocks, **no** Lead fires.
- [ ] Book a test appointment → confirmation + reminders queue.

## Notes

- Only the RV Solar Meta Pixel (`1514677843570094`) loads. GTM was removed from the page
  to kill a duplicate pixel it was firing — do not re-add it.
- The "Are you looking to grow your business?" question is a Yes/No toggle; the answer is
  sent as `looking_to_grow`.
