# CDN (Check Dat Number) — Handoff

Paste this whole file into a new Claude chat to pick up where we left off.
Written 2026-10-05.

## Who / what

- **Product:** Check Dat Number (checkdatnumber.com), owned by Lucrative Strategies L.L.C. (Texas). People share their experiences with a phone number; contributors see other reports about that number.
- **Platform:** built on Base44. Code changes are made by pasting prompts into **Base44's built-in chat**, which edits the app. The code is in GitHub repo `Base-44-Version-02/Base-44-Version-02`, and the owner also has a local checkout at `C:\Users\Rickj\Desktop\Claude Code\check-dat-number`.
- **Working rule:** Base44 never publishes without the owner's approval. Each Base44 result gets pasted back to Claude for review before publishing.
- **Owner's style:** direct answers, no filler, ask before assuming.

## Done and verified (as of 2026-10-05)

1. **Meta Pixel ad-landing fix (published, verified).** `src/lib/sensitiveUrl.js` blocked the pixel on any URL with 7+ digits in a row. Meta ad URLs carry 18-digit `utm_campaign/utm_content/utm_term` IDs, so the pixel never fired on ad landing pages. Fix: an exact allowlist (`AD_TRACKING_PARAMS`: utm_source, utm_medium, utm_campaign, utm_content, utm_term, fbclid, ad_id) skips the digit check. `phone/e164/number/n` still always block. Verified in Meta Test Events: PageView fires on the UTM test URL, and `/Number?phone=…` sends nothing. Events Manager shows the URL without query parameters; that's display-only.
2. **PageView investigation.** PageView already uses the standard `fbq('track','PageView')`; the "Custom event" label in Test Events is cosmetic. PageView fires once per page load and only on `/` (looks like a deliberate Sept 16 rollback). Ads land on `/`, so that's fine. No change made.
3. **Analytics endpoint hardening (published).** The public `logAnalyticsEvent` endpoint now has:
   - a 60 events/min limit per IP+session
   - an 8 KB size limit
   - an event allowlist: LandingPageViewed, SessionStarted, ReturnSession, NumberSubmitted, ContributionStarted, RegistrationStarted, WatchlistAdded, NotificationOpened, CommentCreated
   - phone/email rejection, with utm_* fields exempt
   - query strings stripped from URLs, with UTM values kept as separate fields
   - rejection counts logged without payloads
   
   ContributionCompleted and RegistrationCompleted are recorded server-side and don't go through this endpoint. Pixel verified still firing after publish.
4. **Privacy tests fixed (test files only).** Four stale tests (feed privacy, unlock protection, removed-photo protection, report/billing access rules) now reach their assertions and pass. Verified behavior:
   - only contributors on a number see its reports
   - authors are shown as pseudonyms
   - locations are rounded
   - private fields are never exposed
   - non-contributors get a 403 before any charge
   - removed reports produce no photo links

## Decisions made

- **Apple App Store: abandoned.** Apple rejected under 4.8 (Sign in with Apple), 3.1.1 (Checks bought via Stripe, no In-App Purchase) and 2.1 (UGC visibility). Owner decided not to pursue it. **Don't bring Apple back up.**
- **Names stay visible.** Reports intentionally include the names/aliases the reported person used. That's a product decision.
- **No user blocking.** Posts are shared among all contributors, not owned by one user.
- **No slur filtering.** Screen only threats and contact details.
- **Consent:** US visitors keep pixels **on until they click Decline** (current behavior). Europe stays opt-in.
- **Honor Global Privacy Control:** a browser GPC signal is treated like Decline.
- **Photos go live automatically** (no pre-review), so the Terms wording must change.
- **Mobile Apps section** in the Privacy Policy stays, reworded as conditional ("If we offer mobile apps…").
- **Ad-partner disclosure** must be generic. Don't name Meta or TikTok; the owner may change providers. The TikTok pixel is paused in TikTok's dashboard.
- **Terms acceptance:** existing users are grandfathered, with no re-prompt.
- **Dispute emails:** use the complainant's verified account email (the form already requires sign-in). Pre-fill it read-only; don't use Resend or SendGrid.
- **privacy@ and legal@lucrative-strategies.net** exist and forward to cdn-admin@. Keep them.

## Remaining work: Base44 prompts, in order

Run each one, paste the result back to Claude, and publish only after review.

### 0. Ground rules (paste once at the start of the Base44 chat)
> For all the tasks I'm about to give you: don't change the Meta/TikTok pixel code, pixel events (including CDN_FunnelSubmitted and CompleteRegistration), consent logic, sensitiveUrl.js, the analytics endpoint, or the Stripe checkout. Don't publish anything unless I say so. After each task, run the relevant tests, including the four privacy tests, and report what changed and what passed.

### 1. Privacy Policy and Terms wording + GPC
> Mostly text changes to the Privacy Policy and Terms, plus one small consent change. This task is an exception to my ground rule about consent logic, for the GPC change only.
>
> First, read-only: describe exactly what the cookie banner does today. For a US visitor who hasn't clicked anything, what loads? What does Decline turn off? How is Europe handled? Does the TikTok pixel still load?
>
> Then:
> 1. **GPC:** if the visitor's browser sends a Global Privacy Control signal (`navigator.globalPrivacyControl === true`), treat it exactly like clicking Decline: don't load the ad pixels. Change nothing else about consent. Visitors without the signal behave exactly as today. Add a test for both cases.
> 2. **Privacy Policy:** add that we work with advertising and analytics partners (such as social media and advertising platforms) who may use cookies, pixels, and similar technologies on our site to measure and improve our advertising, and that this may be considered "sharing" for cross-context behavioral advertising under California law. Don't name specific companies. Remove "we don't track you across other companies' apps or websites" and the "There's nothing to opt out of, because it doesn't happen" statement. Describe the cookie banner exactly as it actually works. State that visitors, including California residents, can opt out of this sharing by choosing Decline in the cookie banner, and that we honor Global Privacy Control browser signals as an opt-out.
> 3. **"Mobile Apps and App Store Purchases":** keep the section but make it conditional, e.g. "If we offer mobile apps, purchases made through an app store are processed by Apple or Google…" Don't imply apps exist today.
> 4. **Terms → Image Uploads:** replace "Images that appear on the platform have passed a moderation review for policy compliance" with wording that says images may be reviewed, restricted, or removed by moderators, without claiming every image was reviewed.
> 5. **Terms → Cancellation:** explain that Premium can be cancelled anytime from Wallet → Manage membership without deleting the account, and that account deletion is separate.
> 6. Update the "Last Updated" dates. Show me every change before publishing.

**After publishing:** in Meta Events Manager → Test Events, open
`https://checkdatnumber.com/?utm_source=meta&utm_medium=paid_social&utm_campaign=120247501785380353&utm_content=120248270795540353&utm_term=120247501785380999`
in normal Chrome and confirm PageView still appears.

### 2. Follow-up emails
> Two parts:
> 1. Disputes: send the complainant a confirmation email on submission (with their reference number and what happens next), and an email when the dispute is resolved (outcome only, no details about other users). Require the account email: pre-fill the dispute form's email field with the complainant's verified account email and make it read-only (remove the separate email entry). Update any form text to say confirmation and updates go to their account email. Use the built-in workflow emails.
> 2. Search the whole app for any UI text that promises an email, notification, receipt, or follow-up ("we'll email you," "you'll hear from us," "within X days," etc.). List each one, where it is, and whether something actually sends it. Don't change those yet; report first.
> Add tests for both dispute emails.

(Known issue: the dispute form currently promises follow-up in 3–5 business days, but no email is sent.)

### 3. Dispute removal fix
> The dispute removal action writes "removed" but the supported status is "removed_spam" (or whatever the correct removal status is). Use the correct status, and mark the dispute resolved only after the post update succeeds. Add a test.

### 4. Content screening
> Add server-side screening to report submission, top-level comments, and replies. Screen for threats/violent language and personal contact details (street addresses, phone numbers other than the report's own subject number, email addresses). Do not screen for slurs or profanity.
> - Clean content: publish as today (reports still auto-approve).
> - Flagged content: save it with status "pending_review", hide it from other users, show the author "Your post is being reviewed," and create an Admin Inbox item.
> - A report held for review must not grant contribution access until approved.
> - Replies get the same screening as top-level comments.
> - Admins need Approve and Remove actions on held items.
> Add tests for each category, plus one confirming a clean report still publishes and still grants access.

### 5. Terms acceptance timestamp
> When a user accepts the Terms and Community Rules before posting, store the acceptance with a timestamp and the terms version. Existing users who already accepted are grandfathered in: record their acceptance as "accepted before tracking began" without prompting them again. Enforce on the server that new report and comment submissions require a recorded acceptance. Add a test.

### 6. Moderation check
> The moderation triage workflow has no recorded runs. Submit a test content report, confirm it creates an Admin Inbox item and that the workflow actually runs. Fix whatever's broken. Also confirm "Remove" on a reported item actually hides it from all users. Clean up test data.

## Ads (Meta), in progress

- Sales ad set at **$10/day**. Real sign-ups are about 11–13 new users/day (roughly $0.77–0.91 each). Before the fix, Meta reported far fewer (9 landing-page views from 141 clicks).
- **3–5 days after 2026-10-05:** compare Meta's daily funnel-submitted count (`CDN_FunnelSubmitted`) and landing-page views with real new users. If the gap has narrowed and real sign-ups hold, raise to **$12/day** (20% increase, so learning shouldn't reset). Don't go above $12 until Meta's numbers track reality.
- Ad URL parameters stay as-is: `utm_source=meta&utm_medium=paid_social&utm_campaign={{campaign.id}}&utm_content={{ad.id}}&utm_term={{adset.id}}`. The code fix handles the numeric IDs; editing ads would trigger re-review.

## Legal notes (context, not legal advice)

- Section 230 protects CDN from liability for user posts. It doesn't cover content CDN creates itself, or promises CDN makes and breaks (*Barnes v. Yahoo*), which is why the dispute email and removal fixes matter. A Section 230 repeal bill was introduced in Dec 2025 but hasn't advanced.
- Posters remain liable for false statements of fact. The Privacy Policy already says data may be disclosed under subpoena.
- Suggested: a one-hour consult with a media/internet lawyer on the Terms, the dispute flow, and the pixel disclosure.
