# Monox AI — RV Solar Installer Funnel (v2)

A 2-page lead-gen funnel, static HTML for Vercel. No build step, no frameworks — just
inline CSS/JS plus Google Fonts.

```
/                 index.html        Opt-in page (form gate → /watch)
/watch            watch/index.html  How-it-works video + booking
/favicon.svg      Monox "M" mark
```

**Flow:** Paid Meta traffic lands on `/` → clicks "See How It Works" → fills the modal
form → is redirected to `/watch` with their info + attribution in the query string →
watches the how-it-works video and books a call.

---

## 1. Deploy

1. `git push` this repo to GitHub (already wired to `github.com/stukienukie/rv-solar-funnel`).
2. In Vercel, this is the existing **rv-solar-funnel** project — pushing to the
   production branch redeploys. `index.html` serves at `/`, `watch/index.html` at `/watch`.
3. Custom domain recommended for ad trust (e.g. `go.monox.ai`) → add it in Vercel → Domains.

> This v2 build currently lives on the **`v2-rebuild`** branch. Merge it into the
> production branch (or point Vercel's production branch at it) when you're ready to
> replace the old single-page version.

---

## 2. Fill in the placeholders

Every `{{...}}` token must be replaced before launch. Full list in **§5**. The two files
each open with a placeholder legend in an HTML comment at the top.

Videos:
- **How-it-works (gated)** — a Loom video. Page 1 shows a blurred/locked *teaser* image
  (`{{HOWITWORKS_THUMBNAIL_URL}}`); the real video plays on `/watch` via the Loom embed
  (`{{HOWITWORKS_LOOM_EMBED_URL}}`). Get the embed URL from Loom → Share → Embed.
- **Case study (free)** — one freely-playable video on page 1, HTML5 `<video>`
  (`{{CASESTUDY_MP4_URL}}` + `{{CASESTUDY_POSTER_URL}}`).

---

## 3. GHL Inbound Webhook setup (lead capture)

1. GHL → **Automation → Workflows → New Workflow → trigger "Inbound Webhook."**
2. Copy the webhook URL into `CONFIG.WEBHOOK_URL` at the top of `index.html`'s script.
3. Deploy, submit one real test from `/`, then in the trigger **map the sample payload**.
   The page POSTs this JSON:
   ```json
   {
     "name": "...", "email": "...", "phone": "+15551234567",
     "installs_per_month": "3-5", "page": "rv-solar-funnel",
     "submitted_at": "2026-01-01T00:00:00.000Z",
     "utm_source": "...", "utm_medium": "...", "utm_campaign": "...",
     "utm_content": "...", "utm_term": "...", "fbclid": "...", "fbc": "...", "fbp": "..."
   }
   ```
4. Workflow actions, in order:
   - **Create/Update Contact** (name, email, phone).
   - **Update Contact Field** → custom field `Installs Per Month` = `installs_per_month`.
   - Save `utm_source`, `utm_campaign`, `utm_content`, `fbclid` to contact custom fields
     (create them first if they don't exist).
   - **Add Tag** `rv-solar-funnel-lead`.
   - **Internal Notification** to Luke + Isaac, including the installs value.

> The form **always** redirects to `/watch`, even if the webhook errors or times out
> (5s timeout → one retry → last-resort `no-cors` POST → redirect regardless). A lost
> lead is worse than a duplicate.

---

## 4. Booking automations (separate workflow)

Trigger **"Customer Booked Appointment"** on this funnel's calendar:
- Immediate confirmation **email + SMS**.
- Reminder **24h** before, reminder **1h** before.
- If/Else on appointment status after the slot:
  - **No-show** → SMS: *"Hey {{contact.first_name}}, we missed you — want to grab a new
    time?"* + rebooking link.
  - **Showed** → move to the post-call pipeline stage.

---

## 5. Placeholder checklist

**`index.html`**
- [ ] `{{META_PIXEL_ID}}` — Meta Pixel ID (appears 3×: init, PageView noscript img)
- [ ] `{{GHL_INBOUND_WEBHOOK_URL}}` — CONFIG.WEBHOOK_URL
- [ ] `{{HOWITWORKS_THUMBNAIL_URL}}` — blurred locked teaser image
- [ ] `{{CASESTUDY_POSTER_URL}}` — free case-study video poster
- [ ] `{{CASESTUDY_MP4_URL}}` — free case-study video source
- [ ] `{{TIMEFRAME_1}}` · `{{TIMEFRAME_2}}` · `{{TIMEFRAME_4}}` · `{{TIMEFRAME_5}}`
- [ ] `{{CHRIS_COMPANY}}` — Chris's company (review card)

**`watch/index.html`**
- [ ] `{{META_PIXEL_ID}}` — same pixel ID (appears 2×)
- [ ] `{{HOWITWORKS_LOOM_EMBED_URL}}` — Loom embed URL for the how-it-works video
- [ ] `{{GHL_BOOKING_WIDGET_URL}}` — GHL booking widget URL
- [ ] `{{TIMEFRAME_1}}` · `{{TIMEFRAME_2}}` · `{{TIMEFRAME_4}}` · `{{TIMEFRAME_5}}`

> Tucson RV Solar has no timeframe token — it shows the **MONTH ONE** pill instead.
> The `/watch` aggregate stats band ($250K+ / 1,000+ / 270+) is derived from the five
> case studies; update it if the numbers change.

---

## 6. Test checklist

- [ ] Submit a test on `/` → contact appears in GHL with tag `rv-solar-funnel-lead`,
      the `Installs Per Month` field, and UTM/fbclid fields populated.
- [ ] Redirect lands on `/watch` with `name`, `email`, `phone`, `installs_per_month`
      and attribution params in the URL.
- [ ] Meta **Events Manager → Test Events**:
  - `PageView` on both pages.
  - `ViewContent` when the form modal opens on `/`.
  - `Lead` fires **once** on `/watch`, showing advanced-matching quality (em/ph/fn).
- [ ] Refresh `/watch` → `Lead` does **not** fire again (sessionStorage guard).
- [ ] Temporarily break `CONFIG.WEBHOOK_URL` → confirm the redirect to `/watch` still
      happens.
- [ ] Book a test appointment → confirmation + reminders queue in GHL.

---

## Notes

- **Meta Pixel is hardcoded on-page** for reliability. GTM (`GTM-NKKF36BW`) is also
  loaded, but do **not** re-install the pixel inside GTM or events will double-fire.
- Pixel `Lead` carries an `eventID` (stored in sessionStorage) so server-side CAPI
  dedup can be added later without touching the page.
- Radio buttons (not a `<select>`) are used for "installs per month" — friendlier taps
  for a trades audience; same required-single-choice behavior.
