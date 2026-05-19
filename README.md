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

# 1) Create the task
created = requests.post("https://api.capzy.ai/createTask", json={
    "clientKey": KEY,
    "task": {
        "type": "FunCaptchaTaskProxyLess",
        "websiteURL": "https://example.com",
        "websiteKey": "69A21A01-CC7B-B9C6-0F9A-E7FA06677FFC"
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
    "websiteURL": "https://example.com",
    "websiteKey": "69A21A01-CC7B-B9C6-0F9A-E7FA06677FFC"
  }
}
```

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `type` | `string` | yes | FunCaptchaTaskProxyLess or FunCaptchaTask |
| `websiteURL` | `string` | yes | Full URL of the page |
| `websiteKey` | `string` | yes | The FunCaptcha public key (UUID format). Found in the FunCaptcha init call. |
| `subdomain` | `string` | no | Optional custom API subdomain if the site uses one |
| `proxyType` | `string` | no  | http | https | socks4 | socks5 (only for `FunCaptchaTask`) |
| `proxyAddress` | `string` | no  | IP or hostname of your proxy (only for `FunCaptchaTask`) |
| `proxyPort` | `integer` | no  | Port number of your proxy (only for `FunCaptchaTask`) |
| `proxyLogin` | `string` | no  | Optional — omit if your proxy doesn't require auth (only for `FunCaptchaTask`) |
| `proxyPassword` | `string` | no  | Optional — omit if your proxy doesn't require auth (only for `FunCaptchaTask`) |

Full reference in [`docs/parameters.md`](docs/parameters.md).

## Response shape

When the task is ready (`status: "ready"`), `solution` contains:

| Field | Type | Notes |
|-------|------|-------|
| `token` | `string` | The FunCaptcha token. Submit on the target site exactly where FunCaptcha expects it. |

### How to use the result

Submit the token as the FunCaptcha response field on the target site's form or API.

## Features

- Rotation, matching, and selection puzzles
- Supports custom API subdomains for white-labeled deployments
- High accuracy on standard challenge types

## FAQ

**Where do I find the public key?** Search the page source for `publicKey`, `data-pkey`, or network requests to arkoselabs.com / funcaptcha.com. The key is in UUID format.

**The site passes a `blob` or `data` parameter — do I need it?** Yes — if you see a `data` or `blob` in the FunCaptcha init call, include it as the `subdomain`-adjacent param. Open an issue if you need explicit blob support.

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
