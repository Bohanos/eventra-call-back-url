# Eventra Payment Callback

A single static page that sends users back to the Eventra mobile app after they pay on Paystack.

Paystack only accepts an `https` Callback URL, so it cannot redirect straight to the app's `eventraapp://` link. This page is the `https` address Paystack returns to. It immediately forwards the user to the app and passes along the payment reference.

## How it works

1. A user taps **Pay** in the Eventra app, and the app opens the Paystack checkout in the browser.
2. After paying, Paystack redirects to this page, for example `https://<your-domain>/?trxref=abc123&reference=abc123`.
3. The page's script builds `eventraapp://payment/callback?trxref=abc123&reference=abc123`, keeping the same query string, and opens it.
4. The app opens its `payment/callback` screen, verifies the reference with the backend and shows the ticket.

If the app doesn't open automatically, the user can tap the **Open Eventra** button on the page.

## Files

```
index.html    The whole page (HTML, CSS, script and logo in one file)
README.md     This file
```

There is no build step, no dependencies and no environment variables.

## Deploying on Vercel

1. Push this folder to its own GitHub repository. Keep it separate from the mobile app and web projects.
2. In Vercel, choose **Add New → Project** and import the repository.
3. Leave all settings at their defaults. Framework preset: **Other**. No build command and no output directory.
4. Deploy, then open **Settings → Deployment Protection** and make sure it is **Disabled**. Otherwise Paystack and phones will land on a Vercel login screen.
5. Copy the production URL, for example `https://eventra-pay-callback.vercel.app`.

Every push to the main branch redeploys automatically, and the URL stays the same.

## Paystack settings (done by the Paystack account owner)

In the Paystack dashboard, go to **Settings → API Keys & Webhooks**:

| Setting | Value |
|---|---|
| Callback URL | The production URL of this page |
| Webhook URL | `https://eventra-backend-aidf.onrender.com/api/payments/webhook` |

Set both for **Test** mode while testing and again for **Live** mode when going live. Each mode has its own settings.

The webhook is the safety net. If a user closes the browser before returning to the app, Paystack still tells the backend, and the booking is confirmed.

## Testing

1. Install the latest EAS preview build of the Eventra app. Expo Go cannot receive `eventraapp://` links, so it won't work for this test.
2. Book an event and pay with a Paystack test card.
3. You should land on this page for a moment and then be returned to the app, which shows the ticket.

To test the page on its own, open it with a fake reference:

```
https://<your-domain>/?reference=test123
```

The browser should try to open `eventraapp://payment/callback?reference=test123`. On a phone with the app installed, the app opens.

## Troubleshooting

| Problem | Likely cause |
|---|---|
| Vercel login or "Authentication required" page | Deployment Protection is still on. Disable it. |
| Browser shows "Dangerous site" | A new `vercel.app` domain was flagged by Safe Browsing. Request a review at Google Safe Browsing, or host on Netlify or Cloudflare Pages instead. |
| Page shows but the app doesn't open | The app isn't installed, or the user is in Expo Go. Install the build. The scheme is `eventraapp` in the app's `app.json`. |
| Paystack still returns to an old page | The Callback URL wasn't updated for the right mode (Test or Live). |
| Payment went through but no ticket | Check the backend webhook is set and returns 200. The booking is confirmed from the webhook. |

## Notes

- The page only forwards the query string and does not read or store any payment data.
- If the app's URL scheme (`eventraapp`) or callback route (`payment/callback`) ever changes, update the `link` line in `index.html` to match.

Built by Group 15, Tech Crush Cohort 8.