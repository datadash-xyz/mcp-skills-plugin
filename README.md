# Datadash AI Plugin

Official Datadash plugin for AI coding tools. Turn your AI assistant into a Polymarket analyst: look up
wallets, see how smart money is positioned on an event, find markets where smart money disagrees with the
price, and build wallet cohorts, directly from your editor.

Works with **Claude** (web and desktop), **Claude Code**, **Codex**, and **Gemini CLI**.

---

## Installation

Every client needs a Datadash API key, which is sent to the Datadash MCP server
(`https://api.datadash.xyz/mcp`) as the `X-Api-Key` header.

### Claude (web and desktop)

1. In Claude's plugin settings, add the marketplace `datadash-xyz/mcp-skills-plugin`.
2. Install **Datadash** from it.

This installs the Datadash MCP server and both skills.

### Claude Code

From inside Claude Code:

```
/plugin marketplace add datadash-xyz/mcp-skills-plugin
/plugin install datadash@datadash
/reload-plugins
```

Enter your Datadash API key when prompted.

### Codex

1. Set your API key in the environment Codex runs in:
    ```bash
    export DATADASH_API_KEY=<your-api-key>
    ```

2. Add the marketplace:
    ```bash
    codex plugin marketplace add datadash-xyz/mcp-skills-plugin
    ```

3. Install the plugin from inside Codex:
    ```
    codex
    # Then run /plugins, select Datadash, and install
    /plugins
    ```

### Gemini CLI

```bash
gemini extensions install https://github.com/datadash-xyz/mcp-skills-plugin
```

Enter your Datadash API key when prompted (it is stored as `DATADASH_API_KEY`).

---

## Local development

To work on the plugin from a local checkout in Claude Code:

```bash
git clone https://github.com/datadash-xyz/mcp-skills-plugin
claude --plugin-dir ./mcp-skills-plugin
```

Then run `/mcp` from inside Claude Code and check that `plugin:datadash:datadash` is connected.

---

## What's Inside

| Plugin | Description |
| --- | --- |
| [datadash](./) | The Datadash MCP server plus skills for Polymarket wallet and smart money analysis |

---

## Datadash Plugin

The MCP server exposes Datadash's Polymarket analytics tables and wallet cohorts. The skills teach the
assistant how to use them well: which tables answer which question, what counts as smart money, and how
to read and present the numbers.

### Skills

| Skill | What it does |
| --- | --- |
| `wallet-analysis` | Reports on a wallet or trader: PnL, ROI, win rate and rank over time, strongest categories, open and past positions, recent trades and unusually large bets. Compares wallets side by side and builds wallet cohorts |
| `smart-money` | Shows how smart money is positioned on an event or market and who holds each side, finds markets where smart money's price differs most from the market's, and surfaces insider-like bets |

Skills load automatically when a question matches. In Claude Code you can also call one directly, for
example `/datadash:smart-money`.

### Example prompts

- "How is smart money positioned on the Fed rate decision?"
- "Which markets does smart money disagree with the most right now?"
- "Analyze wallet Theo4." (by display name)
- "What is 0x56687bf447db6ffa42ffe2204a05edaa20f55839 holding right now?" (by address)
- "Compare these three wallets over the last month."
- "Show the biggest insider-like bets in Politics."

By default, smart money means the top 1000 wallets by all-time PnL in the event's two main categories (or
overall, for questions across all markets). Ask for different criteria or name one of your cohorts to
use another wallet set.

---

## Repository Structure

```text
.agents/plugins/
  marketplace.json            # Marketplace catalog (Codex)
.claude-plugin/
  marketplace.json            # Marketplace catalog (Claude Code)
  plugin.json                 # Plugin manifest and MCP server (Claude Code)
.codex-plugin/
  plugin.json                 # Plugin manifest and MCP server (Codex)
gemini-extension.json         # Extension manifest and MCP server (Gemini CLI)
skills/
  smart-money/
  wallet-analysis/
```

---

## Requirements

- **MCP-compatible client**: Claude (web or desktop), Claude Code, Codex, or Gemini CLI
- **Datadash account** with an API key ([datadash.xyz](https://datadash.xyz))
