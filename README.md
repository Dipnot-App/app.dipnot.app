# app.dipnot.app

Static site that serves the **Dipnot** mobile app's universal-link
manifests at `https://app.dipnot.app`. Hosted on Vercel, auto-deploys
on every push to `main`.

This repo does **not** render any user-visible page today. It exists
so iOS and Android can verify that the Dipnot mobile app is the
authorised handler for `https://app.dipnot.app/*` links — which is
what lets taps on a shared article link open the app directly instead
of the browser.

## What lives here

| Path | Purpose |
|---|---|
| [`public/apple-app-site-association`](public/apple-app-site-association) | iOS Universal Links manifest. Declares the app's team id + bundle id as the authorised handler for any path under the domain. |
| [`public/.well-known/assetlinks.json`](public/.well-known/assetlinks.json) | Android App Links manifest. Declares the app's package name + signing-cert SHA-256 as the authorised handler. |
| [`vercel.json`](vercel.json) | Forces `Content-Type: application/json` on both manifest paths. The `apple-app-site-association` file has no extension, so without this override Vercel serves it with an inferred type that iOS rejects. |

## How the link flow works

1. User taps `https://app.dipnot.app/article/<id>` in another app (mail, WhatsApp, browser).
2. The OS checks both manifests (or its cached copy) for a matching handler.
3. If the user has the Dipnot app installed → the OS launches it with the URL, the app's internal router opens the article detail screen.
4. If not → the browser opens `https://app.dipnot.app/article/<id>`. Today this 404s; a static "Download the app" fallback page is planned.

## Changing a manifest

Edits to this repo flow through the Dipnot mobile codebase's change process. Any modification to these files (new SHA fingerprints, additional associated apps, path scoping) must:

1. Be coordinated with the mobile app repo so the bundle id / package name / SHA values stay in lockstep.
2. Land via a PR on `main` (Vercel deploys automatically).

Both manifests are cached aggressively by iOS and Android after an app install. Updates may require end-users to reinstall the app before the new manifest takes effect on their device.

## Adding a new platform

If Dipnot ever ships on a new OS that needs its own well-known file, add it under `public/.well-known/` (or the platform's expected path) and add a matching headers entry to `vercel.json` if the file has no extension.

## License

Proprietary. © Dipnot.
