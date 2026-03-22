# skill

Agent skill for [Citesurf](https://www.citesurf.com), an AI visibility monitoring platform.

Check if ChatGPT, Claude, Perplexity, and Gemini recommend any brand. Get visibility scores, sentiment, competitor data, and actionable insights from your AI agent.

## Install

```bash
npx skills add citesurf/skill
```

Works with Claude Code, Cursor, GitHub Copilot, Gemini CLI, and any agent that supports the [Agent Skills](https://agentskills.io) format.

## Requirements

- A Citesurf account with an active Plus or Max subscription
- An API key (create one in Dashboard > Settings)
- Prepaid credits for scan operations

## What It Does

| Action | Description | Credits |
| --- | --- | --- |
| Create brand | Add a URL to monitor across all 4 AI platforms | 0 |
| Scan | Trigger a scan across ChatGPT, Claude, Gemini, Perplexity | 1 |
| Brand visibility | Check visibility score, sentiment, competitors | 0 |
| Report | Get a full visibility report in one call | 0 |
| Insights | Get AI generated recommendations | 0 |
| Site audit | Check robots.txt, llms.txt, schema.org | 0 |

## Example Prompts

```text
> Check if AI recommends my brand at https://example.com
> Scan https://competitor.com for AI visibility
> What insights does Citesurf have for my brand?
> How visible is my brand on ChatGPT and Gemini?
```

## Documentation

- [Skill definition](skills/citesurf/SKILL.md) for overview and usage
- [API reference](skills/citesurf/references/API.md) for all endpoints with request/response shapes

## License

MIT
