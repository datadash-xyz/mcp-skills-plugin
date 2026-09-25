---
name: wallet-analysis
description: Use when answering questions about Polymarket wallets or smart money with the Datadash MCP tools — how smart money is positioned on an event or market, comparing wallet sets or cohorts, or presenting wallets and their trades to the user. Not needed for plain market or event lookups that mention no wallets.
---

# Wallet analysis with Datadash

Use the Datadash MCP tools. Call get_schema once before composing any query, and keep the result.

## Smart money for an event or market

"Smart money", unless the user gives their own criteria, means the top 1000 wallets in the event's
two main categories over the whole history (duration ALL). Build it the way the DataDash app does:

1. Find the event (a market's row carries its eventId) and read its tagIds.
2. Rank those tags with query_lookup on a tagIds field, filtering id to the event's tagIds and ordering by
   numEvents descending; keep the first two.
3. Restrict the wallets with a userTagProfileLookup filter on the wallet field, for both tags at once:

   ```json
   {"field": "userId", "operator": "userTagProfileLookup", "value": {
     "tagId": {"operator": "in", "value": [<tag1>, <tag2>]},
     "duration": {"operator": "in", "value": ["ALL"]},
     "filter": [{"field": "rank", "operator": "lte", "value": 1000}]}}
   ```

   It goes in walletFilters on unifiedSmartMoneySummary and unifiedSmartMoneyHolders, and in filter on
   signalScore, all queried with query_table. Say which two categories and which duration you used when you
   report the result.
4. Then ask whether the user wants to compare against another wallet set, and offer concrete options: a
   few relevant existing cohorts from list_cohorts, and criteria you think fit the event, such as the
   overall top wallets through userProfileLookup, a shorter duration like 1M, or a minimum roi or
   realizedPnl. Run the comparison with the same query and a different filter; a cohort is applied with
   the inCohort operator.

## Showing wallets

When your output is rendered as Markdown in a terminal (such as Claude Code, Codex CLI or Gemini CLI),
show every wallet you mention as a Markdown link to its DataDash page:
`[label](https://datadash.xyz/wallet/<address>)`, where `<address>` is the wallet's address (the user id)
and the label is the user's display name if known, otherwise the shortened address. For example:
[0x5e5f…7a4f](https://datadash.xyz/wallet/0x5e5f614738782747709a8b5e1f4c4456e5bf7a4f).
