# Parameters reference — FunCaptcha (Arkose Labs)

Every field you can pass to `POST /createTask` for this task type.

## Envelope

```json
{
  "clientKey": "capzy_xxxxxxxxxxxxxxxxxxxxxxxx",
  "task": { ... }
}
```

| Field        | Required | Notes                                                       |
|--------------|:--------:|-------------------------------------------------------------|
| `clientKey`  | yes      | Your Capzy API key. Starts with `capzy_`. Find it at [capzy.ai/dashboard/api-keys](https://capzy.ai/dashboard/api-keys). |
| `task`       | yes      | The task object — see below.                                |

## Task object

### Required + optional fields

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `type` | `string` | yes | `FunCaptchaTaskProxyLess` (we supply the IP) or `FunCaptchaTask` (you supply the IP — fields below) |
| `websiteURL` | `string` | yes | Full URL of the page that loads the FunCaptcha widget. |
| `websiteKey` | `string` | yes | Arkose public key (UUID format). Open DevTools → Network on the target page, look for the Arkose api.js URL — it's `https://<subdomain>.arkoselabs.com/v2/<your-pkey>/api.js`. The 36-character UUID in the path **is** the public key. |
| `funcaptchaApiJSSubdomain` | `string` | recommended | The subdomain that hosts Arkose's `api.js` for your target site. Find it in the api.js URL on the target page: `https://<subdomain>.arkoselabs.com/v2/<your-pkey>/api.js` — pass the `<subdomain>.arkoselabs.com` part. Defaults to `client-api.arkoselabs.com` if omitted, but most production deployments use a different subdomain so set this explicitly. |
| `data` | `string` | conditional | Session-scoped blob the target site's frontend generates and passes to `Arkose.setConfig({ data: { blob: ... } })`. **Required by most production Arkose deployments** — without it, the challenge always fails regardless of how well we solve it. We can't generate this (it's signed against the customer's authentic session + a tenant-specific Arkose key). See [`docs/blob-extraction.md`](blob-extraction.md) for the step-by-step DevTools capture guide. Pass as a plain string. |


### Proxy fields (only for `FunCaptchaTask`)

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `proxyType` | `string` | no | http | https | socks4 | socks5 |
| `proxyAddress` | `string` | no | IP or hostname of your proxy |
| `proxyPort` | `integer` | no | Port number of your proxy |
| `proxyLogin` | `string` | no | Optional — omit if your proxy doesn't require auth |
| `proxyPassword` | `string` | no | Optional — omit if your proxy doesn't require auth |


## Response

### `POST /createTask` success

```json
{
  "errorId": 0,
  "taskId":  "12345"
}
```

### `POST /getTaskResult` while processing

```json
{
  "errorId": 0,
  "status":  "processing"
}
```

### `POST /getTaskResult` when ready

```json
{
  "errorId":  0,
  "status":   "ready",
  "solution": { ... }
}
```

The `solution` object contains:

| Field | Type | Notes |
|-------|------|-------|
| `token` | `string` | The FunCaptcha token. Submit on the target site exactly where FunCaptcha expects it. |

### How to use the solution

Submit the token as the FunCaptcha response field on the target site's form or API.

### Error

```json
{
  "errorId":          1,
  "errorCode":        "ERROR_KEY_DOES_NOT_EXIST",
  "errorDescription": "Invalid API key"
}
```

`errorId` is `0` on success, `1` on any error. The `errorCode` is the
stable machine-readable identifier. Common codes:

- `ERROR_KEY_DOES_NOT_EXIST` — bad API key
- `ERROR_NO_BALANCE` — account balance below the cost of this task
- `ERROR_INVALID_PARAMS` — missing required field or malformed value
- `ERROR_MAX_TASKS_REACHED` — concurrent in-flight cap reached (default 30)
- `ERROR_RATE_LIMITED` — too many createTask calls per second
- `ERROR_TIMEOUT` — solve took longer than the cap (auto-refunded)
- `ERROR_CAPTCHA_UNSOLVABLE` — solver gave up (auto-refunded)
- `ERROR_FUNCAPTCHA_PARAMS_MISSING` — Arkose iframe never bound. The error description tells you which param is most likely missing — usually the `data` blob, or the wrong `funcaptchaApiJSSubdomain`. Refunded.

## Naming conventions

Field names are camelCase on the wire (`websiteURL`, `websiteKey`,
`proxyAddress`). Stick to that exactly when you build the JSON.
