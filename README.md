<div align="center">

<img src="https://capzy.ai/capzy-logo.svg" alt="Capzy" width="220" />

# FunCaptcha (Arkose Labs) Solver

**Solve FunCaptcha rotation, matching, and selection puzzles.**

[![Solve cost](https://img.shields.io/badge/from-%240.001%20%2F%20solve-%23ff5d2a)](https://capzy.ai/pricing)
[![Speed](https://img.shields.io/badge/avg%20solve-~10%20seconds-%2322c55e)](https://capzy.ai/products/funcaptcha)
[![Uptime](https://img.shields.io/badge/uptime-99.9%25-%2322c55e)](https://capzy.ai/status)
[![License: MIT](https://img.shields.io/badge/license-MIT-%23ff5d2a)](LICENSE)

[Live Demo](https://capzy.ai/products/funcaptcha/demo) ·
[Get Free $0.10 Credit](https://capzy.ai/auth/register) ·
[Dashboard](https://capzy.ai/dashboard) ·
[Full Docs](https://capzy.ai/docs) ·
[Pricing](https://capzy.ai/pricing)

</div>

---

## What this repo is

Copy-pasteable examples for solving **FunCaptcha (Arkose Labs)** through the
[Capzy](https://capzy.ai) HTTP API — no SDK required. Pure curl, Python,
and Node.js using the raw API. Easy to read, easy to port, easy to audit.

## What is FunCaptcha (Arkose Labs)?

FunCaptcha (Arkose Labs) presents interactive puzzles: rotating 3D objects to match a target, selecting matching images, or identifying scene objects. One of the harder captchas to solve automatically due to its visual complexity. Capzy handles all standard puzzle variants.

## Why Capzy

- **From $0.001 per solve.** Flat pricing — no tiers, no retainer, no monthly minimum.
- **~10 seconds average solve.** Production-grade speed.
- **Drop-in compatible.** `createTask` / `getTaskResult` protocol. If your code already speaks the standard solver shape, swap the host to `https://api.capzy.ai`.
- **$0.10 in real credits on sign-up.** No card. 100 free test solves.

## Pricing

| Task type | When to use | Cost / solve |
|-----------|-------------|-------------:|
| `FunCaptchaTaskProxyLess`             | Proxyless (Capzy supplies the IP) | **$0.001**   |
| `FunCaptchaTask`                       | You supply the proxy              | **$0.001**   |

For consistency across the target site, use the proxy variant with the
**same proxy your session is already running through** — the solver
mints the token from that IP, so when you submit it back through the
same proxy everything looks consistent.

## 60-second quickstart

```bash
# 1. Sign up — gets you $0.10 in free credits (100 solves)
open https://capzy.ai/auth/register

# 2. Copy your API key from the dashboard
#    https://capzy.ai/dashboard/api-keys

# 3. Run any example
export CAPZY_KEY="capzy_..."
bash examples/curl/basic.sh
```

Minimal Python:

```python
import requests, time

KEY = "capzy_xxxxxxxxxxxxxxxxxxxxxxxx"

# 1) Create the task. Most production sites require both the api.js
#    subdomain AND a session-scoped `data` blob captured from your
#    authentic browser session — see docs/blob-extraction.md for the
#    step-by-step DevTools walkthrough.
created = requests.post("https://api.capzy.ai/createTask", json={
    "clientKey": KEY,
    "task": {
        "type": "FunCaptchaTaskProxyLess",
        "websiteURL": "https://example.com/signup",
        "websiteKey": "<YOUR-PUBLIC-KEY>",
        # The subdomain that hosts api.js on YOUR target page. Find it
        # in the script URL: https://<this-part>.arkoselabs.com/v2/<pkey>/api.js
        "funcaptchaApiJSSubdomain": "<your-subdomain>.arkoselabs.com",
        # The blob from the target site's setupEnforcement callback.
        # Required by most production Arkose deployments.
        "data": "<blob-from-target-site>",
    },
}).json()
task_id = created["taskId"]

# 2) Poll until ready
while True:
    result = requests.post("https://api.capzy.ai/getTaskResult", json={
        "clientKey": KEY, "taskId": task_id,
    }).json()
    if result["status"] == "ready":
        break
    time.sleep(2)

print(result["solution"])
```

That's the whole protocol. The rest of this repo is just that, in every
language we could think of.

## Pick your language

| Language        | Example                                       |
|-----------------|-----------------------------------------------|
| **curl / bash** | [`examples/curl/basic.sh`](examples/curl/basic.sh)    |
| **Python**      | [`examples/python/basic.py`](examples/python/basic.py) |
| **Node.js**     | [`examples/nodejs/basic.js`](examples/nodejs/basic.js) |

See [`examples/README.md`](examples/README.md) for setup details.

## Request envelope

```json
{
  "clientKey": "capzy_xxxxxxxxxxxxxxxxxxxxxxxx",
  "task": {
    "type": "FunCaptchaTaskProxyLess",
    "websiteURL": "https://example.com/signup",
    "websiteKey": "<YOUR-PUBLIC-KEY>",
    "funcaptchaApiJSSubdomain": "<your-subdomain>.arkoselabs.com",
    "data": "<blob-from-target-site>"
  }
}
```

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `type` | `string` | yes | `FunCaptchaTaskProxyLess` (we supply the IP) or `FunCaptchaTask` (you supply it) |
| `websiteURL` | `string` | yes | Full URL of the page that loads the FunCaptcha widget |
| `websiteKey` | `string` | yes | Arkose public key (UUID). Find it in the api.js URL on the target page: `https://<subdomain>.arkoselabs.com/v2/<your-pkey>/api.js`. |
| `funcaptchaApiJSSubdomain` | `string` | recommended | The subdomain that hosts Arkose's `api.js` for your target. Find it the same way as the public key — in the api.js URL. Defaults to `client-api.arkoselabs.com` if omitted; most production sites use a different subdomain so set this explicitly. |
| `data` | `string` | **conditional** | Session-scoped blob the target site's frontend generates. Required by most production Arkose deployments. We can't generate this — capture it from your authentic browser session. See [`docs/blob-extraction.md`](docs/blob-extraction.md) for the step-by-step. |
| `proxyType` | `string` | no  | `http` / `https` / `socks4` / `socks5` (only for `FunCaptchaTask`) |
| `proxyAddress` | `string` | no  | IP or hostname of your proxy (only for `FunCaptchaTask`) |
| `proxyPort` | `integer` | no  | Port number of your proxy (only for `FunCaptchaTask`) |
| `proxyLogin` | `string` | no  | Omit if your proxy doesn't require auth (only for `FunCaptchaTask`) |
| `proxyPassword` | `string` | no  | Omit if your proxy doesn't require auth (only for `FunCaptchaTask`) |

Full reference in [`docs/parameters.md`](docs/parameters.md). Blob extraction walkthrough in [`docs/blob-extraction.md`](docs/blob-extraction.md).

## Response shape

When the task is ready (`status: "ready"`), `solution` contains:

| Field | Type | Notes |
|-------|------|-------|
| `token` | `string` | The FunCaptcha token. Submit on the target site exactly where FunCaptcha expects it. |

### How to use the result

Submit the token as the FunCaptcha response field on the target site's form or API.

## Features

- Rotation, matching, and selection puzzles
- Customer-supplied subdomain + blob for full per-site control
- High accuracy on standard challenge types

## FAQ

**Where do I find the public key?** Open DevTools → Network on the target page and look for the Arkose `api.js` request — its URL is `https://<subdomain>.arkoselabs.com/v2/<your-pkey>/api.js`. The 36-char UUID in the path is the public key. You can also grep the page source for `publicKey:` or `data-pkey=`.

**Do I need the `data` blob?** Most production Arkose deployments require it. If your first solve returns `ERROR_FUNCAPTCHA_PARAMS_MISSING`, the blob is almost always why. We can't mint the blob — only your authentic browser session can. See [`docs/blob-extraction.md`](docs/blob-extraction.md) for the capture walkthrough.

**Do I need `funcaptchaApiJSSubdomain`?** Yes for almost all production sites. Find it in the api.js script URL on the target page — pass the `<subdomain>.arkoselabs.com` part. The default `client-api.arkoselabs.com` works for a minority of deployments.

## What you'll need

- A Capzy API key — [sign up](https://capzy.ai/auth/register) (free, $0.10 credit).
- Network access to `https://api.capzy.ai`.

## Other captcha types

Capzy solves 25+ captcha types. Full catalog at
[capzy.ai/pricing](https://capzy.ai/pricing). Each type has its own
solver repo on [github.com/capzy-ai](https://github.com/capzy-ai).

## License

[MIT](LICENSE).

---

<div align="center">

**[Sign up for free credits →](https://capzy.ai/auth/register)**

Built by [Capzy](https://capzy.ai). Issues + PRs welcome.

</div>
