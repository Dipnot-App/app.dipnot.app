# app.dipnot.app

Static site that serves the **Dipnot** mobile app's universal-link
manifests at `https://app.dipnot.app`. Hosted on Vercel, auto-deploys
on every push to `main`.

It exists first so iOS and Android can verify that the Dipnot mobile
app is the authorised handler for `https://app.dipnot.app/*` links —
which is what lets taps on a shared article link open the app directly
instead of the browser. It also serves the small pages a visitor sees
when that hand-off doesn't happen (see below).

## What lives here

| Path | Purpose |
|---|---|
| [`public/apple-app-site-association`](public/apple-app-site-association) | iOS Universal Links manifest. Declares the app's team id + bundle id as the authorised handler for any path under the domain. |
| [`public/.well-known/assetlinks.json`](public/.well-known/assetlinks.json) | Android App Links manifest. Declares the app's package name + signing-cert SHA-256 as the authorised handler. |
| [`public/article.html`](public/article.html) | Fallback for `/article/<id>`: "Uygulamada aç" plus the platform's store button. Carries the link's `utm_*` params into the app and the stores. |
| [`public/index.html`](public/index.html), [`public/yarisma-2026.html`](public/yarisma-2026.html) | Root page (app or store) and the competition's shareable page. |
| [`vercel.json`](vercel.json) | Forces `Content-Type: application/json` on both manifest paths. The `apple-app-site-association` file has no extension, so without this override Vercel serves it with an inferred type that iOS rejects. |

## How the link flow works

1. User taps `https://app.dipnot.app/article/<id>` in another app (mail, WhatsApp, browser).
2. The OS checks both manifests (or its cached copy) for a matching handler.
3. If the user has the Dipnot app installed → the OS launches it with the URL, the app's internal router opens the article detail screen.
4. If not → the browser opens `https://app.dipnot.app/article/<id>`, which `vercel.json` rewrites to `article.html`. The same happens for installed users in in-app browsers that ignore Universal Links (Instagram, X), and for long-pressed or pasted links.
5. On a phone, `article.html` tries `dipnotapp://article/<id>` and shows "Uygulamada aç" plus that platform's store button. On a desktop it shows both store buttons.

### Campaign tags (`utm_*`)

Shared links may carry `utm_source`, `utm_medium` and `utm_campaign` (vocabulary agreed with the admin panel; DipnotAPP #742). `article.html` keeps only those three and passes them on:

- **Into the app**: appended to the `dipnotapp://` deep link, where the article screen adds them to `article_open`.
- **Google Play**: URL-encoded into `&referrer=`. Firebase Analytics reads the Install Referrer on first open, so `first_open` carries the campaign in GA.
- **App Store**: `?ct=<utm_campaign>` (or `utm_source` when there is no campaign), 40 characters max. Visible only in App Store Connect, not in GA.

The page has no analytics of its own: adding GA here would raise a cookie-consent question.

## Changing a manifest

Edits to this repo flow through the Dipnot mobile codebase's change process. Any modification to these files (new SHA fingerprints, additional associated apps, path scoping) must:

1. Be coordinated with the mobile app repo so the bundle id / package name / SHA values stay in lockstep.
2. Land via a PR on `main` (Vercel deploys automatically).

Both manifests are cached aggressively by iOS and Android after an app install. Updates may require end-users to reinstall the app before the new manifest takes effect on their device.

## Adding a new platform

If Dipnot ever ships on a new OS that needs its own well-known file, add it under `public/.well-known/` (or the platform's expected path) and add a matching headers entry to `vercel.json` if the file has no extension.

## License

Proprietary. © Dipnot.
