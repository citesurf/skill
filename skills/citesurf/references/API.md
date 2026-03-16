# Citesurf API Reference

Base URL: `https://citesurf.com/api/v1`

All requests require: `Authorization: Bearer $CITESURF_API_KEY`

## Account

### Get Account Info

`GET /account`

Returns plan, payment status, credit balance, brand count, and brand limit.

```json
{
  "data": {
    "plan": "PLUS",
    "paymentStatus": "ACTIVE",
    "brandCount": 1,
    "brandLimit": 1,
    "creditBalance": 12
  }
}
```

## Brands

### List Brands

`GET /brands`

Returns all monitored brands with latest visibility metrics.

```json
{
  "data": [
    {
      "id": "cm...",
      "name": "Acme",
      "website": "https://acme.com",
      "type": "COMPANY",
      "category": "Project Management Tool",
      "description": "Acme helps teams plan and track work.",
      "language": "en",
      "createdAt": "2026-01-15T10:30:00.000Z",
      "visibilityScore": 42,
      "mentionRate": 0.65,
      "scoreChange": 0,
      "probeCount": 48,
      "scanActive": true,
      "lastScanAt": "2026-03-14T08:00:00.000Z"
    }
  ]
}
```

### Get Brand

`GET /brands/{brandId}`

Returns detailed brand data including competitors, platform breakdown, and sentiment.

```json
{
  "data": {
    "id": "cm...",
    "name": "Acme",
    "website": "https://acme.com",
    "type": "COMPANY",
    "category": "Project Management Tool",
    "description": "Acme helps teams plan and track work.",
    "language": "en",
    "fixedPrompts": [
      "best project management tools",
      "is Acme worth it",
      "Acme vs Monday"
    ],
    "visibilityScore": 42,
    "mentionRate": 0.65,
    "scoreChange": 5,
    "averagePosition": 3.2,
    "shareOfVoice": 0.18,
    "caveatRate": 0.12,
    "avgFirstMentionOffset": 0.25,
    "sentimentDistribution": {
      "positive": 0.6,
      "neutral": 0.3,
      "negative": 0.1
    },
    "topCitedDomains": [
      { "domain": "acme.com", "count": 12 },
      { "domain": "g2.com", "count": 8 }
    ],
    "platformBreakdown": [
      { "platform": "CHATGPT", "score": 55, "mentionRate": 55, "probeCount": 48 },
      { "platform": "PERPLEXITY", "score": 48, "mentionRate": 48, "probeCount": 48 },
      { "platform": "CLAUDE", "score": 35, "mentionRate": 35, "probeCount": 48 },
      { "platform": "GEMINI", "score": 30, "mentionRate": 30, "probeCount": 48 }
    ],
    "competitors": [
      { "name": "Monday.com", "website": "https://monday.com", "mentionCount": 24 },
      { "name": "Asana", "website": "https://asana.com", "mentionCount": 18 }
    ],
    "scanActive": true,
    "scanCount": 8,
    "probeCount": 48,
    "lastScanAt": "2026-03-14T08:00:00.000Z"
  }
}
```

Note: `platformBreakdown` is sorted by score descending. `competitors` shows top 20 sorted by mention count.

### Create Brand

`POST /brands`

Body:

```json
{
  "name": "Acme",
  "website": "https://acme.com",
  "language": "en"
}
```

- `name`: 1 to 100 characters
- `website`: valid URL (automatically normalized)
- `language` (optional, default `"en"`): `en`, `es`, `de`, `fr`, `pt`, `it`, `nl`, `pl`

AI analyzes the site, detects brand type, generates personas and prompts, and triggers the first scan. Rate limited: 10 per 24 hours. Brand limits: Plus = 1, Max = 5.

```json
{
  "data": {
    "id": "cm...",
    "name": "Acme"
  }
}
```

### Update Brand

`PATCH /brands/{brandId}`

Body (all fields optional, at least one required):

```json
{
  "type": "COMPANY",
  "category": "Project Management Tool",
  "description": "Acme helps teams plan and track work with AI powered insights.",
  "prompts": ["best project management tools", "is Acme worth it", "Acme vs Monday"]
}
```

Constraints:

- `type`: `PERSON`, `PRODUCT`, `COMPANY`, or `SHOP`
- `category`: max 100 characters
- `description`: max 500 characters
- `prompts`: exactly 3 strings, 5 to 200 characters each

Changes take effect on the next scan.

```json
{
  "data": {
    "id": "cm...",
    "name": "Acme",
    "website": "https://acme.com",
    "type": "COMPANY",
    "category": "Project Management Tool",
    "description": "Acme helps teams plan and track work with AI powered insights.",
    "language": "en",
    "prompts": ["best project management tools", "is Acme worth it", "Acme vs Monday"],
    "updatedAt": "2026-03-16T12:00:00.000Z"
  }
}
```

### Delete Brand

`DELETE /brands/{brandId}`

Archives the brand and stops monitoring. Removes scan schedule.

```json
{
  "data": {
    "success": true
  }
}
```

## Scans

### List Scans

`GET /brands/{brandId}/scans?offset=0&pageSize=20`

Paginated list of scans with aggregate metrics per scan.

Query parameters:

- `offset` (default: 0)
- `pageSize` (default: 20, max: 100)

```json
{
  "data": {
    "scans": [
      {
        "id": "cm...",
        "createdAt": "2026-03-14T08:00:00.000Z",
        "visibilityScore": 42,
        "mentionRate": 0.65,
        "averagePosition": 3.2,
        "platformScores": {
          "CHATGPT": 55,
          "CLAUDE": 35,
          "PERPLEXITY": 48,
          "GEMINI": 30
        },
        "sentimentDistribution": {
          "positive": 0.6,
          "neutral": 0.3,
          "negative": 0.1
        },
        "topCompetitors": [
          { "name": "Monday.com", "mentions": 6 }
        ],
        "topCitedDomains": [
          { "domain": "acme.com", "count": 4 }
        ],
        "probeCount": 48,
        "shareOfVoice": 0.18,
        "caveatRate": 0.12,
        "avgFirstMentionOffset": 0.25
      }
    ],
    "hasMore": true,
    "offset": 0,
    "pageSize": 20,
    "total": 8
  }
}
```

### Get Scan Detail

`GET /brands/{brandId}/scans/{scanId}`

Full scan detail including all probes with per platform results. Site audit is included only if an audit was performed within one day of the scan.

```json
{
  "data": {
    "id": "cm...",
    "createdAt": "2026-03-14T08:00:00.000Z",
    "visibilityScore": 42,
    "mentionRate": 0.65,
    "averagePosition": 3.2,
    "platformScores": { "CHATGPT": 55, "CLAUDE": 35, "PERPLEXITY": 48, "GEMINI": 30 },
    "sentimentDistribution": { "positive": 0.6, "neutral": 0.3, "negative": 0.1 },
    "topCompetitors": [{ "name": "Monday.com", "mentions": 6 }],
    "topCitedDomains": [{ "domain": "acme.com", "count": 4 }],
    "probeCount": 48,
    "shareOfVoice": 0.18,
    "caveatRate": 0.12,
    "avgFirstMentionOffset": 0.25,
    "probes": [
      {
        "id": "cm...",
        "promptText": "best project management tools",
        "isFixed": true,
        "persona": { "name": "Startup Founder" },
        "platforms": {
          "CHATGPT": {
            "mentioned": true,
            "position": 2,
            "sentiment": "POSITIVE",
            "sentimentScore": 0.85,
            "contextType": "RECOMMENDATION",
            "recommendationStrength": "strong",
            "competitorsMentioned": ["Monday.com", "Asana"],
            "response": "Here are the best project management tools...",
            "citations": [
              { "domain": "acme.com", "url": "https://acme.com/features", "position": 1 }
            ]
          },
          "CLAUDE": {
            "mentioned": false,
            "position": null,
            "sentiment": null,
            "sentimentScore": null,
            "contextType": null,
            "recommendationStrength": null,
            "competitorsMentioned": ["Monday.com"],
            "response": "For project management, I'd suggest...",
            "citations": null
          }
        }
      }
    ],
    "siteAudit": {
      "checkedAt": "2026-03-14T08:05:00.000Z",
      "robotsTxt": {
        "found": true,
        "allowsGPTBot": true,
        "allowsClaudeBot": false,
        "allowsPerplexityBot": true,
        "allowsGoogleExtended": true
      },
      "llmsTxt": {
        "found": true,
        "valid": true,
        "contentLength": 2048
      },
      "schema": {
        "found": true,
        "types": ["Organization", "SoftwareApplication"]
      },
      "sitemap": {
        "found": true,
        "urlCount": 156,
        "lastmod": "2026-03-10T00:00:00.000Z"
      },
      "openGraph": {
        "found": true,
        "hasTitle": true,
        "hasDescription": true,
        "hasImage": true,
        "hasUrl": true,
        "hasType": true
      }
    }
  }
}
```

### Trends

`GET /brands/{brandId}/trends?range=30`

Historical visibility trends. Range options: `7`, `30`, `90` days (default: 30).

```json
{
  "data": {
    "trends": [
      {
        "date": "Mar 14",
        "visibilityScore": 42,
        "mentionRate": 0.65,
        "averagePosition": 3.2,
        "CHATGPT": 55,
        "CLAUDE": 35,
        "PERPLEXITY": 48,
        "GEMINI": 30
      }
    ],
    "range": 30
  }
}
```

### Trigger Scan

`POST /brands/{brandId}/scan`

Costs 1 credit. Scans all 4 AI platforms. No request body required. Rate limited: 10 per hour. If the scan fails to trigger, the credit is automatically refunded.

```json
{
  "data": {
    "brandId": "cm...",
    "creditsUsed": 1,
    "creditsRemaining": 11,
    "status": "triggered"
  }
}
```

## Prompts

### Get Prompts

`GET /brands/{brandId}/prompts?platform=CHATGPT&offset=0&pageSize=10`

AI platform probe results grouped by prompt. What each AI said about the brand.

Query parameters:

- `platform` (optional): `CHATGPT`, `CLAUDE`, `PERPLEXITY`, `GEMINI`
- `offset` (default: 0)
- `pageSize` (default: 10, max: 100)

```json
{
  "data": {
    "prompts": [
      {
        "id": "best project management tools",
        "text": "best project management tools",
        "isFixed": true,
        "persona": { "name": "Startup Founder" },
        "scans": [
          {
            "id": "cm...",
            "platform": "CHATGPT",
            "mentioned": true,
            "position": 2,
            "sentiment": "POSITIVE",
            "sentimentScore": 0.85,
            "sentimentConfidence": "HIGH",
            "sentimentNormalized": 0.9,
            "contextType": "RECOMMENDATION",
            "recommendationStrength": "strong",
            "competitorsMentioned": ["Monday.com"],
            "competitorDensity": 0.4,
            "competitiveContext": "comparative_list",
            "responseStructure": "numbered_list",
            "responseLength": 1200,
            "responseHash": "a1b2c3...",
            "mentionCount": 1,
            "firstMentionOffset": 0.15,
            "listSize": 8,
            "hasCaveats": false,
            "caveats": [],
            "audienceFits": ["startups", "small teams"],
            "conditionalTriggers": [],
            "response": "Here are the best project management tools...",
            "createdAt": "2026-03-14T08:00:00.000Z"
          }
        ],
        "stats": {
          "totalScans": 4,
          "mentionRate": 75,
          "avgPosition": 2.5,
          "platformBreakdown": {
            "CHATGPT": { "mentioned": true, "position": 2, "sentiment": "POSITIVE" },
            "CLAUDE": { "mentioned": false, "position": null, "sentiment": null },
            "PERPLEXITY": { "mentioned": true, "position": 3, "sentiment": "POSITIVE" },
            "GEMINI": { "mentioned": true, "position": 4, "sentiment": "NEUTRAL" }
          }
        },
        "totalResults": 12,
        "hasMore": true
      }
    ],
    "count": 1,
    "platformFilter": "CHATGPT",
    "hasMore": false,
    "offset": 0,
    "pageSize": 10
  }
}
```

Note: `mentionRate` in stats is a percentage (0 to 100). `platformFilter` is `"all"` when no filter is applied.

## Personas

### Get Personas

`GET /brands/{brandId}/personas?offset=0&pageSize=2`

Probe results grouped by persona, showing how different user types discover the brand.

Query parameters:

- `offset` (default: 0)
- `pageSize` (default: 2, max: 50)

```json
{
  "data": {
    "personas": [
      {
        "id": "cm...",
        "name": "Startup Founder",
        "description": "Early stage founder evaluating tools for a growing team.",
        "prompts": [
          {
            "id": "best project management tools",
            "text": "best project management tools",
            "scans": [
              {
                "id": "cm...",
                "platform": "CHATGPT",
                "mentioned": true,
                "position": 2,
                "sentiment": "POSITIVE",
                "contextType": "RECOMMENDATION",
                "recommendationStrength": "strong",
                "response": "Here are the best project management tools...",
                "createdAt": "2026-03-14T08:00:00.000Z"
              }
            ],
            "totalResults": 4,
            "hasMore": true
          }
        ]
      }
    ],
    "hasMore": true,
    "offset": 0,
    "pageSize": 2
  }
}
```

## Insights

### Get Insights

`GET /brands/{brandId}/insights?status=pending&offset=0&pageSize=20`

AI generated recommendations for improving visibility, prioritized by impact.

Query parameters:

- `status` (optional): `pending`, `completed`
- `offset` (default: 0)
- `pageSize` (default: 20, max: 50)

```json
{
  "data": {
    "insights": [
      {
        "id": "cm...",
        "brandId": "cm...",
        "type": "VISIBILITY",
        "priority": "HIGH",
        "title": "Unblock ClaudeBot in robots.txt",
        "description": "Your robots.txt currently blocks ClaudeBot. Claude cannot crawl your site for training data.",
        "action": "Add 'Allow: /' for ClaudeBot in your robots.txt file.",
        "confidence": "HIGH",
        "metadata": null,
        "createdAt": "2026-03-14T08:10:00.000Z",
        "updatedAt": "2026-03-14T08:10:00.000Z",
        "completedAt": null,
        "improvement": null
      }
    ],
    "hasMore": false,
    "offset": 0,
    "pageSize": 20,
    "counts": {
      "all": 5,
      "pending": 3,
      "completed": 2
    }
  }
}
```

Insight types: `VISIBILITY`, `SENTIMENT`, `POSITIONING`, `COMPETITION`, `OPPORTUNITY`.

Priority levels: `HIGH`, `MEDIUM`, `LOW`.

Confidence levels: `HIGH`, `MEDIUM`, `LOW`.

### Update Insight

`PATCH /brands/{brandId}/insights/{insightId}`

Body:

```json
{ "action": "dismiss" }
```

or

```json
{ "action": "complete" }
```

**Dismiss** deletes the insight and returns:

```json
{
  "data": {
    "success": true
  }
}
```

**Complete** marks the insight as completed and validates improvement with before/after scan metrics:

```json
{
  "data": {
    "insight": { "..." : "full insight object with completedAt and improvement set" },
    "validation": {
      "improvement": { "..." : "before/after metrics" },
      "message": "Visibility improved by 8 points since this insight was created."
    }
  }
}
```

If the insight is already completed, it returns the existing insight unchanged.

## Site Audit

### Get Site Audit

`GET /brands/{brandId}/site-audit`

Technical audit results for the brand's website. Returns `null` in data if no audit exists.

Checks: robots.txt (AI crawler access), llms.txt (presence and validity), schema.org JSON-LD, XML sitemap (freshness), Open Graph completeness.

```json
{
  "data": {
    "checkedAt": "2026-03-14T08:05:00.000Z",
    "robotsTxt": {
      "found": true,
      "allowsGPTBot": true,
      "allowsClaudeBot": false,
      "allowsPerplexityBot": true,
      "allowsGoogleExtended": true
    },
    "llmsTxt": {
      "found": true,
      "valid": true,
      "contentLength": 2048
    },
    "schema": {
      "found": true,
      "types": ["Organization", "SoftwareApplication"]
    },
    "sitemap": {
      "found": true,
      "urlCount": 156,
      "lastmod": "2026-03-10T00:00:00.000Z"
    },
    "openGraph": {
      "found": true,
      "hasTitle": true,
      "hasDescription": true,
      "hasImage": true,
      "hasUrl": true,
      "hasType": true
    }
  }
}
```

## Report

### Get Report

`GET /brands/{brandId}/report`

Comprehensive visibility report in a single call. 0 credits. Use this when you need a complete overview instead of calling multiple endpoints separately.

Returns brand info, visibility scores, platform breakdown, top prompts (up to 3 fixed and 3 dynamic, sorted by mention rate), sentiment and context distribution, competitors (top 10), cited domains (top 5), trends (last 10 scans), site audit, audit issues, and top 3 pending insights.

```json
{
  "data": {
    "brandName": "Acme",
    "website": "https://acme.com",
    "description": "Acme helps teams plan and track work.",
    "language": "en",
    "summary": {
      "visibilityScore": 42,
      "mentionRate": 65,
      "averagePosition": 3.2,
      "totalScans": 48,
      "scoreChange": 5,
      "shareOfVoice": 0.18,
      "caveatRate": 0.12,
      "avgFirstMentionOffset": 0.25
    },
    "platformScores": {
      "CHATGPT": 55,
      "CLAUDE": 35,
      "PERPLEXITY": 48,
      "GEMINI": 30
    },
    "prompts": [
      {
        "text": "best project management tools",
        "category": "Fixed",
        "personaName": "Startup Founder",
        "mentionRate": 75,
        "totalResponses": 4,
        "mentions": 3,
        "platformResults": {
          "CHATGPT": { "mentioned": true, "position": 2, "sentiment": "POSITIVE" },
          "CLAUDE": { "mentioned": false, "position": null, "sentiment": null },
          "PERPLEXITY": { "mentioned": true, "position": 3, "sentiment": "POSITIVE" },
          "GEMINI": { "mentioned": true, "position": 4, "sentiment": "NEUTRAL" }
        }
      }
    ],
    "sentimentDistribution": {
      "positive": 0.6,
      "neutral": 0.3,
      "negative": 0.1
    },
    "contextDistribution": {
      "RECOMMENDATION": 12,
      "COMPARISON": 8,
      "WARNING": 1,
      "NEUTRAL": 3
    },
    "competitors": [
      { "name": "Monday.com", "mentionCount": 6 },
      { "name": "Asana", "mentionCount": 4 }
    ],
    "citedDomains": [
      { "domain": "acme.com", "count": 12 },
      { "domain": "g2.com", "count": 8 }
    ],
    "trends": [
      {
        "date": "2026-03-14T08:00:00.000Z",
        "visibilityScore": 42,
        "mentionRate": 0.65,
        "platformScores": { "CHATGPT": 55, "CLAUDE": 35, "PERPLEXITY": 48, "GEMINI": 30 }
      }
    ],
    "siteAudit": {
      "robotsTxt": { "found": true, "allowsGPTBot": true, "allowsClaudeBot": false, "allowsPerplexityBot": true, "allowsGoogleExtended": true },
      "llmsTxt": { "found": true, "valid": true, "contentLength": 2048 },
      "schema": { "found": true, "types": ["Organization"] },
      "sitemap": { "found": true, "urlCount": 156, "lastmod": "2026-03-10T00:00:00.000Z" },
      "openGraph": { "found": true, "hasTitle": true, "hasDescription": true, "hasImage": true, "hasUrl": true, "hasType": true }
    },
    "auditIssues": [
      { "type": "CRAWLER_BLOCKED_CLAUDEBOT", "priority": "HIGH" }
    ],
    "insights": [
      {
        "type": "VISIBILITY",
        "priority": "HIGH",
        "title": "Unblock ClaudeBot in robots.txt",
        "description": "Your robots.txt currently blocks ClaudeBot.",
        "action": "Add 'Allow: /' for ClaudeBot in your robots.txt file."
      }
    ]
  }
}
```

Note: `siteAudit` and `auditIssues` are omitted when no audit exists. `mentionRate` in prompts is a percentage (0 to 100). Report trends use ISO date strings (unlike the trends endpoint which uses short format like "Mar 14").

## Credit System

Each scan costs 1 credit and runs across all 4 platforms (ChatGPT, Claude, Gemini, Perplexity). Reading data never costs credits. Brand creation is free. Purchase credit packs at [citesurf.com](https://citesurf.com).

## Rate Limits

| Action | Limit | Window |
| --- | --- | --- |
| Brand creation | 10 | 24 hours |
| Scan trigger | 10 | 1 hour |

Exceeding a limit returns `429` with a `Retry-After` header (in seconds).

## Error Codes

| Code | Status | Meaning |
| --- | --- | --- |
| `AUTH_REQUIRED` | 401 | Missing or invalid API key |
| `FORBIDDEN` | 403 | Subscription cancelled or insufficient permissions |
| `VALIDATION_FAILED` | 400 | Invalid request body |
| `INSUFFICIENT_CREDITS` | 402 | Not enough credits for scan |
| `NOT_FOUND` | 404 | Resource not found |
| `RATE_LIMITED` | 429 | Rate limit exceeded |
| `SERVER_ERROR` | 500 | Internal server error |

Validation errors include a `details` object with per field messages:

```json
{
  "error": "VALIDATION_FAILED",
  "message": "Validation failed",
  "details": {
    "name": ["String must contain at least 1 character(s)"],
    "website": ["Invalid website"]
  }
}
```

Insufficient credits errors include the current balance:

```json
{
  "error": "INSUFFICIENT_CREDITS",
  "message": "Insufficient credits",
  "balance": 0
}
```

Rate limit errors include a retry message:

```json
{
  "error": "RATE_LIMITED",
  "message": "Rate limit exceeded. Try again in 42 second(s)."
}
```
