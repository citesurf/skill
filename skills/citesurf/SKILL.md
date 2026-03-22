---
name: citesurf
description: Check if AI recommends any brand. Scan URLs for AI visibility across ChatGPT, Claude, Gemini, and Perplexity. Get visibility scores, sentiment analysis, competitor data, and actionable insights via the Citesurf API.
license: MIT
metadata:
  author: citesurf
  version: "1.0"
---

# Citesurf AI Visibility Monitoring

Check if ChatGPT, Claude, Gemini, and Perplexity recommend a brand. Get visibility scores, sentiment, competitors, and what to fix first.

## Authentication

All requests require an API key in the `Authorization` header:

```text
Authorization: Bearer $CITESURF_API_KEY
```

Get your API key at [citesurf.com](https://www.citesurf.com) > Dashboard > Settings.
Requires an active Plus or Max subscription and prepaid credits for scans.

Base URL: `https://www.citesurf.com/api/v1`

## Recommended Workflow

1. **Check account** to verify subscription and credit balance before triggering scans
2. **Find or create a brand** by listing existing brands or creating one from a URL
3. **Trigger a scan** (1 credit) to scan all 4 AI platforms
4. **Read results** using the report endpoint for a full overview, or individual endpoints for specific data

## Check Account

Verify subscription status and credit balance before triggering scans:

```bash
curl https://www.citesurf.com/api/v1/account \
  -H "Authorization: Bearer $CITESURF_API_KEY"
```

Returns plan (PLUS or MAX), payment status, credit balance, brand count, and brand limit.

## Create a Brand

Add a new URL to monitor. AI analyzes the site, generates personas and prompts, and triggers the first scan automatically:

```bash
curl -X POST https://www.citesurf.com/api/v1/brands \
  -H "Authorization: Bearer $CITESURF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Acme", "website": "https://acme.com", "language": "en"}'
```

Supported languages: `en`, `es`, `de`, `fr`, `pt`, `it`, `nl`, `pl`.

Rate limited: 10 per 24 hours. Brand limits depend on plan (Plus: 1, Max: 5).

## Check Brand Visibility

For brands already being monitored:

```bash
# List all monitored brands
curl https://www.citesurf.com/api/v1/brands \
  -H "Authorization: Bearer $CITESURF_API_KEY"

# Get detailed brand data
curl https://www.citesurf.com/api/v1/brands/{brandId} \
  -H "Authorization: Bearer $CITESURF_API_KEY"
```

Brand detail includes: visibility score, score change, mention rate, platform breakdown (per platform scores), sentiment distribution, competitors, average position, share of voice, and caveat rate.

## Trigger a Scan

Scan all 4 AI platforms for an existing brand. Costs 1 credit:

```bash
curl -X POST https://www.citesurf.com/api/v1/brands/{brandId}/scan \
  -H "Authorization: Bearer $CITESURF_API_KEY"
```

No request body required. Rate limited: 10 per hour. Returns credits used and remaining balance. If the scan fails to trigger, the credit is automatically refunded.

## Scans and Trends

```bash
# Paginated scan list
curl "https://www.citesurf.com/api/v1/brands/{brandId}/scans?offset=0&pageSize=20" \
  -H "Authorization: Bearer $CITESURF_API_KEY"

# Full scan detail with probes and citations
curl https://www.citesurf.com/api/v1/brands/{brandId}/scans/{scanId} \
  -H "Authorization: Bearer $CITESURF_API_KEY"

# Visibility trends over time (7, 30, or 90 days)
curl "https://www.citesurf.com/api/v1/brands/{brandId}/trends?range=30" \
  -H "Authorization: Bearer $CITESURF_API_KEY"
```

## Prompts and Personas

```bash
# What each AI platform said about the brand, grouped by prompt
curl https://www.citesurf.com/api/v1/brands/{brandId}/prompts \
  -H "Authorization: Bearer $CITESURF_API_KEY"

# Probe results grouped by persona (how different user types discover the brand)
curl https://www.citesurf.com/api/v1/brands/{brandId}/personas \
  -H "Authorization: Bearer $CITESURF_API_KEY"
```

Optional prompt filter: `?platform=CHATGPT`, `CLAUDE`, `PERPLEXITY`, or `GEMINI`.

## Insights

AI generated recommendations for improving visibility, prioritized by impact:

```bash
# List insights
curl https://www.citesurf.com/api/v1/brands/{brandId}/insights \
  -H "Authorization: Bearer $CITESURF_API_KEY"

# Dismiss or complete an insight
curl -X PATCH https://www.citesurf.com/api/v1/brands/{brandId}/insights/{insightId} \
  -H "Authorization: Bearer $CITESURF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"action": "complete"}'
```

Optional query parameter: `?status=pending` or `?status=completed`.

- `dismiss` deletes the insight
- `complete` validates improvement with before/after metrics

## Site Audit

Technical checks for brands with their own domain:

```bash
curl https://www.citesurf.com/api/v1/brands/{brandId}/site-audit \
  -H "Authorization: Bearer $CITESURF_API_KEY"
```

Checks: robots.txt (AI crawler access), llms.txt, schema.org JSON-LD, XML sitemap freshness, Open Graph completeness.

## Update Brand

Correct AI generated metadata such as type, category, description, or the 3 fixed monitoring prompts:

```bash
curl -X PATCH https://www.citesurf.com/api/v1/brands/{brandId} \
  -H "Authorization: Bearer $CITESURF_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"category": "Project Management Tool", "prompts": ["best project management tools", "is Acme PM worth it", "Acme PM vs Monday"]}'
```

All fields optional, only pass what you want to change. Valid types: PERSON, PRODUCT, COMPANY, SHOP. Prompts must be exactly 3 strings (5 to 200 chars each). Changes take effect on the next scan.

## Delete Brand

Archive a brand and stop monitoring:

```bash
curl -X DELETE https://www.citesurf.com/api/v1/brands/{brandId} \
  -H "Authorization: Bearer $CITESURF_API_KEY"
```

## Full Report

Get a comprehensive visibility report in one call. Includes all scores, platform breakdown, top prompts, sentiment, competitors, cited domains, trends, site audit, and insights:

```bash
curl https://www.citesurf.com/api/v1/brands/{brandId}/report \
  -H "Authorization: Bearer $CITESURF_API_KEY"
```

Use this endpoint when you need a complete overview rather than fetching individual endpoints separately. Costs 0 credits.

## Response Format

All responses follow this structure:

- Success: `{ "data": { ... } }`
- Error: `{ "error": "ERROR_CODE", "message": "Human-readable message" }`

Common error codes: `AUTH_REQUIRED` (401), `FORBIDDEN` (403), `NOT_FOUND` (404), `VALIDATION_FAILED` (400), `INSUFFICIENT_CREDITS` (402), `RATE_LIMITED` (429), `SERVER_ERROR` (500).

When rate limited, the response includes a `Retry-After` header in seconds.

## When to Use This Skill

- User asks "is my brand visible on AI?" or "do AI chatbots recommend me?"
- User wants to monitor a brand for AI visibility
- User asks about AI search optimization (AEO/GEO)
- User wants to compare their brand's AI presence across platforms
- User needs insights on improving AI recommendations

## Full API Reference

See [references/API.md](references/API.md) for complete endpoint documentation including request/response shapes, pagination, and all error codes.
