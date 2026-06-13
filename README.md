# safeshelf-web

Static site for [safeshelf.app](https://safeshelf.app) — hosts **Apple Universal Links** and **Android App Links** verification files for the SafeShelf mobile app, plus a lightweight web fallback for magic-link verification.

Hosted on **GitHub Pages** from this repository.

## Files

| Path | Purpose |
|------|---------|
| `.well-known/apple-app-site-association` | iOS Universal Links (AASA) |
| `.well-known/assetlinks.json` | Android App Links |
| `CNAME` | Custom domain for GitHub Pages |
| `auth/verify/index.html` | Browser fallback when the app is not installed |
| `terms/index.html` | Terms of Service (`https://safeshelf.app/terms`) |
| `privacy/index.html` | Privacy Policy (`https://safeshelf.app/privacy`) |
| `assets/legal.css` | Shared styles for legal pages |

## App identifiers

- **Domain:** `safeshelf.app`
- **iOS bundle ID:** `com.safeshelf.app`
- **Android package:** `com.safeshelf.app`
- **Deep link paths:** `/auth/verify`, `/auth/*`

## Before go-live: replace placeholders

### Apple Team ID (AASA)

1. Open [Apple Developer → Membership](https://developer.apple.com/account) and copy your **Team ID** (10 characters).
2. In `.well-known/apple-app-site-association`, replace `YOUR_APPLE_TEAM_ID` so `appID` becomes `{TEAM_ID}.com.safeshelf.app`.
3. Commit and push to `main`.

### Android SHA-256 certificate fingerprint (assetlinks.json)

1. **Debug (local testing):**  
   `keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android`  
   Copy the **SHA256** line (colons optional; Google accepts both formats in many cases — use the format your Play Console / signing docs specify).

2. **Release:** Use the **App signing key certificate** SHA-256 from [Google Play Console](https://play.google.com/console) → Your app → Setup → App integrity, or from your upload/release keystore via `keytool`.

3. In `.well-known/assetlinks.json`, replace `YOUR_SHA256_FINGERPRINT` with that fingerprint (you can add multiple entries in the array for debug + release).

4. Commit and push to `main`.

## Namecheap DNS (safeshelf.app)

Point the apex domain at GitHub Pages. In Namecheap → Domain List → **safeshelf.app** → **Advanced DNS**:

### Apex (`@`)

Add **A records** (GitHub Pages; verify current IPs in [GitHub docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)):

| Type | Host | Value |
|------|------|-------|
| A | @ | `185.199.108.153` |
| A | @ | `185.199.109.153` |
| A | @ | `185.199.110.153` |
| A | @ | `185.199.111.153` |

### `www` (optional)

| Type | Host | Value |
|------|------|-------|
| CNAME | www | `safe-shelf.github.io` |

Or CNAME `www` → `safeshelf.app` if you prefer www on the same apex setup.

After DNS propagates, in the GitHub repo **Settings → Pages**, set custom domain to `safeshelf.app` and enable **Enforce HTTPS** when available.

## GitHub Pages

- **Source:** branch `main`, folder `/` (root)
- **Custom domain:** `safeshelf.app` (via `CNAME` in repo + Pages settings)

## Validation

After deploy and DNS propagation:

```bash
# AASA (no file extension; should be application/json)
curl -sI https://safeshelf.app/.well-known/apple-app-site-association

curl -s https://safeshelf.app/.well-known/apple-app-site-association | jq .

# Android asset links
curl -sI https://safeshelf.app/.well-known/assetlinks.json

curl -s https://safeshelf.app/.well-known/assetlinks.json | jq .

# Fallback page
curl -sI https://safeshelf.app/auth/verify/

# Legal pages
curl -sI https://safeshelf.app/terms/

curl -sI https://safeshelf.app/privacy/

# Verify terms_version meta tag
curl -s https://safeshelf.app/terms/ | grep 'safeshelf:terms-version'
```

Apple: [AASA Validator](https://search.developer.apple.com/appsearch-validation-tool/)  
Google: [Statement List Generator and Tester](https://developers.google.com/digital-asset-links/tools/generator)

## Mobile app repo

Universal / App Link handling and magic-link verify flow live in the SafeShelf **mobile** app (Expo Router route `auth/verify`, deep link config in `app.config.ts`). Keep `associatedDomains` (iOS) and intent filters (Android) aligned with the paths in this repo.

## License

Public configuration for SafeShelf product domain verification.
