# Archonics MCP Audit Server

Free-tier context engineering audits for production AI agents, delivered as MCP tools you can call from Claude Desktop, Cursor, Claude Code, or any MCP-compatible client.

**What you get:** top-3 findings on your system prompts, tool definitions, or context packing, on demand, no account needed.

**What it costs:** nothing. The free scan is genuinely free. Upgrade paths to the $49 Instant Audit and $750 Full Audit are surfaced in the response footer; they're not paywalls on this tool.

## Why this exists

Most production agent failures aren't model failures — they're context engineering failures. Ambiguous instructions, underspecified tools, bloated context, no regression tests on prompt changes. Those problems are spottable by a trained reader. Archonics has trained that reader and published it as an MCP tool so you can get a second opinion on your agent's context without filing a support ticket.

The underlying audit engine applies Archonics Audit Methodology v1.0, the same spec that drives our paid audits.

## Tools

### `audit_system_prompt`

Paste a system prompt. Get back the three most important context engineering issues in it, ranked by severity, with specific recommendations.

**Covers:** role clarity, instruction conflicts, negative space, priority structure when instructions conflict, token efficiency, format specification precision, failure-mode coverage.

### `audit_tool_definition`

Paste a tool/function definition. Get back the three most important issues affecting how reliably the model will call it.

**Covers:** description quality (the "when to use this tool" question), parameter schema precision, parameter documentation, error response design, discoverability.

### `audit_context_packing`

Paste a representative context payload (or describe it structurally). Get back the three most important efficiency and quality issues.

**Covers:** content inventory, redundancy across sections, freshness/relevance, ordering, truncation risk, prompt-cache utilization.

## Installation

### Claude Desktop

Add to your `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "archonics-audit": {
      "command": "npx",
      "args": ["-y", "@archonics/mcp-audit"],
      "env": {
        "ANTHROPIC_API_KEY": "your-anthropic-api-key-here"
      }
    }
  }
}
```

### Cursor

Add to your `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "archonics-audit": {
      "command": "npx",
      "args": ["-y", "@archonics/mcp-audit"],
      "env": {
        "ANTHROPIC_API_KEY": "your-anthropic-api-key-here"
      }
    }
  }
}
```

### Claude Code

```bash
claude mcp add archonics-audit npx -y @archonics/mcp-audit
```

Then set `ANTHROPIC_API_KEY` in your environment.

## Why does it need my Anthropic API key?

The audit engine runs on Claude. You bring your own API key so:

1. Audit submissions go directly from your machine to Anthropic's API, never through Archonics servers.
2. Your costs are transparent — a typical audit uses 2,000–4,000 tokens, well under a penny.
3. There's no "free but actually limited" rate-limit surprise. Your API key, your limits.

If you'd rather not bring your own key, use the $49 Instant Audit at agent.market — we cover the API costs and return a full-methodology audit PDF.

## Privacy

Submitted content is processed ephemerally. No prospect content is retained on Archonics infrastructure or used to train any model. The API call pattern is: your client → your Anthropic API key → Anthropic → your client. Archonics servers are not in this path.

Aggregated, anonymized patterns across many audits may inform improvements to the methodology — "18 of 20 audited systems lacked prompt-regression tests" — but specific content never feeds that process.

Details: [archonics.ai/privacy](https://archonics.ai/privacy)

## Upgrade paths

If the free scan surfaces issues worth fixing, two paid tiers go deeper:

- **Instant Audit — $49 USDC via x402.** Full methodology applied programmatically to a system you submit. 5-10 page PDF report covering all four dimensions (prompt, tools, context, eval) rather than just three findings in one dimension. Listed at [agent.market/archonics](https://agent.market/archonics).
- **Full Audit — $750.** Human-reviewed audit of a complete agent system. 15-25 page report tuned to your team's context. Contact audits@archonics.ai.

## Contact

- Questions, feedback, or false-positive reports: audits@archonics.ai
- Methodology and full audit examples: [archonics.ai](https://archonics.ai)
- Issues with this MCP server: [github.com/archonics/mcp-audit/issues](https://github.com/archonics/mcp-audit)

## License

MIT. Use it, fork it, audit yourself.
