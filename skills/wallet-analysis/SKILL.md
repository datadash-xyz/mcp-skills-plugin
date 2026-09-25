---
name: wallet-analysis
description: Use when answering questions about specific Polymarket wallets or traders with the Datadash MCP tools — a wallet's performance, PnL, win rate, strongest categories, open or past positions and recent trades, comparing several wallets, or building and comparing wallet cohorts. For how smart money is positioned on an event or market, or which markets smart money favors, use the smart-money skill instead.
---

# Wallet analysis with Datadash

Use the Datadash MCP tools. Call get_schema once before composing any query, and keep the result; do not
call it again in the same task.

## Identify the wallet

- A `0x…` address is the wallet's userId. Use it lowercased.
- A display name, pseudonym or X handle is not an id. Resolve it with query_lookup on a table's userId field
  (for example `userProfile`, field `userId`) with `request.searchKey` set to the name, read the returned
  rows (displayName, pseudonym, xUsername, pnl, rank) and use the chosen row's `userId`. If several rows plausibly match, show them and ask which one
  was meant rather than picking one.
- If the user refers to "this wallet" without giving one, ask for the address or name.

## Build a wallet report

Pick the parts the question needs; for an open-ended "analyze this wallet", cover all of them.

1. **Performance** — `userProfile` with `"filter": [{"field": "userId", "operator": "in", "value": [...]}]`
   and a single `duration` (one of `1D`, `1W`, `1M`, `1Y`, `ALL`). Duration is not returned in the rows, so
   query each duration in its own call. Report realizedPnl, roi, winRate, volume, profitFactor,
   numPredictions, holdTime and rank. Put ALL and 1M side by side to show whether the wallet is still
   performing recently; no row for a duration means no activity in it.
2. **Categories** — `userTagProfile` for the wallet with duration `ALL`, ordered by realizedPnl descending.
   Resolve the tagIds to labels with query_lookup on `tagId` so you can name the categories. Name the
   categories it is best and worst in and its rank inside its best one.
3. **Open positions** — `activePosition` (requires a userId filter), ordered by value descending. Show the
   market, side, shares, avgBuyPrice, current price, value and unrealizedPnl.
4. **History** — `closedPosition` or `allPosition` (grouped by event), ordered by pnl, to find the biggest
   wins and losses.
5. **Recent trades** — `activity` with the wallet in a `wallet` filter and a `timestamp` filter such as
   `{"operator": "last", "value": {"length": 30, "unit": "day"}}`, ordered by timestamp descending. Use
   `usdAmount` for trade size.
6. **Unusual bets** — `signalScore` filtered to the wallet shows its currently held positions scored for
   insider-like conviction. Mention positions with a high score or relSize well above 1 (bigger than the
   wallet usually bets).

## Reading the numbers

- `roi` and `winRate` are percentages already (2.34 means 2.34%), not fractions.
- `holdTime` is in seconds; convert it to hours or days.
- `rank` is by realizedPnl for that duration; lower is better.
- A wallet with a huge numTrades, a win rate near 50% and a low roi is most likely a market maker or bot,
  not a directional trader. Say so, because its positions say little about conviction.
- A wallet with few predictions and a very high roi may be lucky on a handful of bets; mention the sample
  size.
- Values are USD. Round money sensibly ($1.2M, $45.3K) and show prices in cents (62¢).

## Compare wallets

Query the same tables with every wallet in one `in` filter, rather than one call per wallet, and put the
results in a single table with one row per wallet. Use the same duration for all of them and say which
one it is.

## Cohorts

A cohort is a saved wallet set. `list_cohorts` shows the existing ones, and any userId or wallet filter
accepts `{"operator": "inCohort", "value": "<cohortId>"}`. To create a cohort from criteria (for example
top 500 in Politics over 1M with roi above 20), read the create_cohort tool's description and
the table's `cohorts` block in the schema. Confirm the criteria with the user before creating, updating
or deleting a cohort, because those are saved to their account.

## Showing wallets

When your output is rendered as Markdown (Claude on the web and desktop, Claude Code, Codex CLI, Gemini
CLI), show every wallet you mention as a Markdown link to its DataDash page:
`[label](https://datadash.xyz/wallet/<address>)`, where `<address>` is the wallet's address (the user id)
and the label is the user's display name if known, otherwise the shortened address. For example, the
wallet `0x56687bf447db6ffa42ffe2204a05edaa20f55839` is shown as
[Theo4](https://datadash.xyz/wallet/0x56687bf447db6ffa42ffe2204a05edaa20f55839) when its display name is
known, and as [0x5668…5839](https://datadash.xyz/wallet/0x56687bf447db6ffa42ffe2204a05edaa20f55839)
otherwise.
