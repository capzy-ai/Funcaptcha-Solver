# Capturing the FunCaptcha `data` blob from DevTools

Many Arkose Labs deployments gate token issuance behind a session-scoped
`blob` value the target site's own frontend generates and passes to
`Arkose.setConfig({ data: { blob: ... } })`.

**The blob can only come from your authentic browser session on the
target site.** We can't mint it — it's signed against your active
session cookies AND a tenant-specific key that only Arkose and the
target site share. If you don't pass it on a site that requires it,
the iframe never binds and the solver returns
`ERROR_FUNCAPTCHA_PARAMS_MISSING`.

## Step-by-step capture

The blob is regenerated on every page load and **expires the moment
the captcha iframe binds**. Capture it BEFORE the iframe loads — if
you refresh the page after the captcha has appeared, you'll need to
refresh again to get a fresh blob.

1. **Open DevTools** (`F12` or `Cmd+Opt+I`) on the page where the
   FunCaptcha widget loads.

2. **Switch to the Network tab.** Make sure recording is on (the red
   circle). Tick **Preserve log** so navigations don't clear the list.

3. **Refresh the page** (`Ctrl+R` / `Cmd+R`). This re-triggers the
   captcha setup script and re-generates a fresh blob.

4. **Filter for `gt2`** in the network filter box. You're looking for
   a request shaped like:

   ```
   POST https://<subdomain>.arkoselabs.com/fc/gt2/public_key/<your-pkey>
   ```

   Where `<subdomain>` is whatever the api.js script URL on the target
   page uses, and `<your-pkey>` is the public key.

5. **Click the request → Payload tab.** The body is form-encoded.
   Find the `data[blob]` field — the value next to it is what you need.

   It's typically a long base64-ish string, often 200-500 characters,
   sometimes JSON-wrapped.

6. **Copy the entire `data[blob]` value** and pass it as the `data`
   parameter on `createTask`:

   ```json
   {
     "clientKey": "capzy_xxx",
     "task": {
       "type": "FunCaptchaTaskProxyLess",
       "websiteURL": "https://target.example.com/signup",
       "websiteKey": "<YOUR-PUBLIC-KEY>",
       "data": "<your-blob-here>"
     }
   }
   ```

## Common gotchas

**The blob is base64-ish but isn't pure base64.** Don't decode it; pass
it verbatim as a string. We accept the JSON-stringified `{"blob":"..."}`
shape too if your client wraps it.

**The blob expires fast.** Once the captcha iframe loads on the target
page, the blob is consumed. If you wait > ~30 seconds between
generating it and calling `createTask`, your solve may fail with
`ERROR_FUNCAPTCHA_PARAMS_MISSING`. Generate a fresh blob right before
each solve.

**The blob is per-session.** A blob generated in one browser session
can't be replayed in another. If you're orchestrating multi-account
flows, each account session needs its own blob.

**Programmatic capture.** If you're running headless / Playwright /
Puppeteer for the rest of your flow, intercept the
`/fc/gt2/public_key/` request on your own browser before any captcha
JS executes, pull `data[blob]` from the form payload, then pause your
script, call Capzy, and resume with the token.

## Auto-detection of `funcaptchaApiJSSubdomain`

You almost never need to pass `funcaptchaApiJSSubdomain` — Capzy
auto-detects the subdomain based on your `websiteURL` host for the
major Arkose deployments. If your target is a regional or niche
deployment that's not auto-detected, pass `funcaptchaApiJSSubdomain`
explicitly (the api.js URL on the target page tells you what to use:
`https://<this-part>.arkoselabs.com/v2/<pkey>/api.js`). The error
message on a failed solve will also tell you which subdomain we tried.
