# datatk-quote-skill

A market data query skill powered by the [dataTrack](https://www.datatk.com) QuoteNode REST API, supporting real-time and historical data across stocks, futures, forex, and more.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
- [Quick Start](#quick-start)
- [Endpoint Overview](#endpoint-overview)
- [Usage Examples](#usage-examples)
- [Market Codes](#market-codes)
- [Error Handling](#error-handling)
- [Reference Docs](#reference-docs)

---

## Prerequisites

1. Install **Node.js 18+** (the script uses the built-in `fetch` — no extra dependencies needed).
2. Go to the [dataTrack service page](https://www.datatk.com/service) to obtain your **Endpoint** and **API Key**.

---

## Configuration

1) Copy the example config file:

```bash
cp env.example.json env.json
```

2) Edit `env.json` in the project root and fill in your endpoint and key:

```json
{
  "endpoint": "https://quote.datatk.com",
  "apiKey": "<YOUR_API_KEY>"
}
```

Notes:
- `env.json` is ignored by git (see `.gitignore`). **Do not commit real API keys.**
- Trailing slashes in `endpoint` are stripped automatically.

---

## Quick Start

All requests go through the unified script `scripts/request.mjs`. Use `--path` for the API path and `--body` for the JSON request body:

```bash
node scripts/request.mjs --path /Api/V1/Quotation/Detail --body '{"instrument":"US|AAPL","lang":"en"}'
```

On success, the script prints the raw JSON response to stdout.  
On failure (HTTP status other than `200`), it prints the status code and error body, then exits.

---

## Security Notes

This skill makes outbound HTTPS requests to QuoteNode REST endpoints and uses an API key.
To reduce the risk of accidental data exfiltration, `scripts/request.mjs` enforces:

- HTTPS-only endpoints
- Domain allowlist (default: `*.datatk.com`)
- Request path must start with `/Api/`
- IP endpoints are rejected

If you use a private gateway domain, update the allowlist inside `scripts/request.mjs`.

---

## Endpoint Overview

| Feature | Path |
| --- | --- |
| Security Detail | `/Api/V1/Quotation/Detail` |
| Instrument List | `/Api/V1/Basic/BasicInfo` |
| Tick Trades | `/Api/V1/Quotation/Tick` |
| Level-2 Depth | `/Api/V1/Quotation/DepthQuoteL2` |
| TimeLine | `/Api/V1/History/TimeLine` |
| KLine | `/Api/V1/History/KLine` |
| Broker List | `/Api/V1/Quotation/Brokers` |
| Broker Queue | `/Api/V1/Quotation/Broker` |
| Trading Calendar | `/Api/V1/Basic/Holiday` |

---

## Usage Examples

### Get US stock quote (Apple)

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/Detail \
  --body '{"instrument":"US|AAPL","lang":"en"}'
```

### Get Hong Kong stock list (page 1, 50 per page)

```bash
node scripts/request.mjs \
  --path /Api/V1/Basic/BasicInfo \
  --body '{"instrument":"HKEX|1","page":1,"pageSize":50,"lang":"en"}'
```

### Get Tencent daily KLine (forward-adjusted)

```bash
node scripts/request.mjs \
  --path /Api/V1/History/KLine \
  --body '{"instrument":"HKEX|00700","period":"day","right":1}'
```

### Get Level-2 depth for a HK stock (5 levels)

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/DepthQuoteL2 \
  --body '{"instrument":"HKEX|00700|R","depth":5}'
```

### Get Hong Kong trading calendar

```bash
node scripts/request.mjs \
  --path /Api/V1/Basic/Holiday \
  --body '{"market":"HKEX","startDate":"2025-01-01","endDate":"2025-12-31","lang":"en"}'
```

### Get tick trades

```bash
node scripts/request.mjs \
  --path /Api/V1/Quotation/Tick \
  --body '{"instrument":"HKEX|00700|R","page":1,"pageSize":20}'
```

---

## Market Codes

### Stock Markets

| Code | Description |
| --- | --- |
| `SSE` | Shanghai Stock Exchange |
| `SZSE` | Shenzhen Stock Exchange |
| `HKEX` | Hong Kong Exchanges and Clearing |
| `US` | US stock market |

### Futures Markets (partial)

| Code | Description |
| --- | --- |
| `SHFE` | Shanghai Futures Exchange |
| `CFFEX` | China Financial Futures Exchange |
| `COMEX` | Commodity Exchange, Inc. |
| `CME` | Chicago Mercantile Exchange |

### Forex

| Code | Description |
| --- | --- |
| `FOREX` | Foreign exchange market |

> For the full list of market codes, security types, KLine periods, and adjustment types, see `references/reference.md`.

---

## instrument Field Format

The `instrument` parameter follows the format `{market}|{symbol}` or `{market}|{symbol}|{flag}`.

| Flag | Description |
| --- | --- |
| `R` | Real-time quote |
| `D` | Delayed quote |

**Examples:**

- `US|AAPL` — Apple (US market)
- `HKEX|00700` — Tencent (HK market)
- `HKEX|00700|R` — Tencent real-time quote
- `COMEX|GC2604` — COMEX gold futures contract

---

## Error Handling

When the HTTP status is not `200`, the response body has this format:

```json
{
  "code": 429,
  "reason": "OPENAPI_QUOTA_EXCEEDED",
  "message": "Rate limit exceeded"
}
```

Common HTTP status codes:

| Status | Description |
| --- | --- |
| `400` | Invalid request parameters |
| `401` | Authentication failed — check your API Key |
| `403` | No permission for this endpoint or market |
| `429` | Rate limit exceeded |
| `500` | Internal server error |

Business error codes (the `code` field in the response body):

| Code | Description |
| --- | --- |
| `0` | Success |
| `1001` | Parameter validation failed |
| `1002` | Instrument not found |
| `2001` | Invalid API Key |
| `2003` | No endpoint permission |
| `2004` | No market permission |

---

## Reference Docs

| Document | Contents |
| --- | --- |
| [references/openapi.md](references/openapi.md) | All endpoint paths, request parameters, and examples |
| [references/reference.md](references/reference.md) | Enum values, market codes, and error codes |
| [references/response.md](references/response.md) | Response field meanings for each endpoint |
| [references/architecture.md](references/architecture.md) | REST layer architecture overview |
