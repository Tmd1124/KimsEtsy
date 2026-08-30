# Social Media Auto-Posting Setup

This app can auto-post the generated landing page copy to Pinterest, Facebook,
Instagram, and TikTok. Each platform needs its own credentials added to `.env`
(copy `.env.example` if you haven't already). Nothing here is required for the
rest of the app (Etsy browsing, landing pages, AI copy) to work — these are
only needed for the "Post to X" buttons.

After adding any of the values below, restart the server (`npm start`).

---

## Pinterest

Status: works via a direct access token (no OAuth flow needed in-app).

1. Go to **developers.pinterest.com/apps** → create an app (Trial/Standard
   access is fine for posting to your own account).
2. Generate an access token for the app (Pinterest's dashboard lets you
   generate one directly for testing — no redirect URI needed for this path).
3. Add to `.env`:
   ```
   PINTEREST_ACCESS_TOKEN=your_token_here
   PINTEREST_APP_ID=your_app_id_here
   ```

**Known blocker:** new Pinterest apps often sit in "trial access pending"
and every `v5` API call returns `"Your application consumer type is not
supported, please contact support"` (error code 3) until Pinterest manually
activates the app. This is on Pinterest's side, not something fixable from
our code. To unblock: post in Pinterest's developer community
(**community.pinterest.biz** → Developers category) with a new topic titled
something like "Trial access activation request — App ID `<your app id>`
(consumer type not supported)". Reference the app ID. Turnaround is usually
hours to a few days.

Once activated, no code changes are needed — the board picker and "Post to
Pinterest" button will just start working.

---

## Facebook + Instagram

Status: uses the same Meta app and a directly-pasted long-lived token (no
OAuth flow needed in-app).

1. Go to **developers.facebook.com** → create a **Business** type app.
2. Use the **Graph API Explorer** (under Tools) to generate a long-lived
   **Page Access Token** for a Facebook Page you manage. Request these
   permissions on the token:
   - `pages_manage_posts`
   - `pages_read_engagement`
   - `instagram_basic`
   - `instagram_content_publish`
3. The Facebook Page must have an Instagram **Business or Creator** account
   linked to it (Page Settings → Linked Accounts) for Instagram posting to
   work — Instagram posting goes through the linked Page's token, not a
   separate Instagram login.
4. Get the **Page ID** (Page Settings → About, or via the Graph API
   Explorer) and the **Instagram Business Account ID** (Graph API Explorer:
   `GET /{page-id}?fields=instagram_business_account`).
5. Add to `.env`:
   ```
   FACEBOOK_PAGE_ACCESS_TOKEN=your_long_lived_page_token_here
   FACEBOOK_PAGE_ID=your_page_id_here
   INSTAGRAM_BUSINESS_ACCOUNT_ID=your_ig_business_account_id_here
   ```

---

## TikTok

Status: requires a full OAuth login flow (per-account, not a pasted token)
plus a video file per post — TikTok's Content Posting API is video-only, it
cannot publish a static product photo.

1. Go to **developers.tiktok.com** → create an app → request the
   **Content Posting API** product (`video.publish` scope).
2. TikTok's web OAuth requires an **HTTPS** redirect URI — plain
   `http://localhost` is not accepted (unlike Pinterest). Easiest fix:
   run `ngrok http 3000` and use the HTTPS URL it gives you.
3. In the TikTok app's settings, set the **Redirect URI** to:
   ```
   https://<your-ngrok-subdomain>.ngrok-free.app/auth/tiktok/callback
   ```
4. Add to `.env`:
   ```
   TIKTOK_CLIENT_KEY=your_client_key_here
   TIKTOK_CLIENT_SECRET=your_client_secret_here
   TIKTOK_REDIRECT_URI=https://<your-ngrok-subdomain>.ngrok-free.app/auth/tiktok/callback
   ```
5. With ngrok running and the server restarted, open
   `http://localhost:3000/auth/tiktok` (or click "Connect TikTok →" on a
   generated TikTok social post) to log in and authorize the app.

**Important limitation:** unaudited TikTok apps can only publish videos as
**private** (`SELF_ONLY` — visible only to the TikTok account that
authorized the app). To post publicly, TikTok has to review and approve the
app for the Content Posting API in production, which is a separate,
longer process on their side.

---

## Where things live in the app

| Platform | Endpoint(s) | Frontend button |
|---|---|---|
| Pinterest | `GET /api/pinterest/boards`, `POST /api/pinterest/pin` | Board picker + "Post to Pinterest" |
| Facebook | `POST /api/facebook/post` | "Post to Facebook Page" |
| Instagram | `POST /api/instagram/post` | "Post to Instagram" |
| TikTok | `GET /auth/tiktok`, `GET /auth/tiktok/callback`, `GET /api/tiktok/status`, `POST /api/tiktok/post` | "Connect TikTok" → file picker + "Post to TikTok" |

All of these already render on the generated landing page based on which
platform is selected in the **Social Media** dropdown before clicking
**Create Landing Page**. Missing credentials just disable the relevant
button with an explanatory message — nothing else in the app is affected.
