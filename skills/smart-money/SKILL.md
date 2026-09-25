---
name: smart-money
description: Use when answering Polymarket smart money questions with the Datadash MCP tools — how smart money is positioned on an event or market, which wallets hold each side, which markets smart money favors against the market price, or which open bets look like insider signals. For questions about one specific wallet's performance or history, use the wallet-analysis skill instead.
---

# Smart money with Datadash

Use the Datadash MCP tools. Call get_schema once before composing any query, and keep the result; do not
call it again in the same task.

## Who counts as smart money

Unless the user gives their own criteria, smart money is a top-ranked wallet set over the whole history
(duration `ALL`), applied as a lookup filter on the wallet field:

- **For one event or market** — the top 1000 wallets in the event's two main categories:

  ```json
  {"field": "userId", "operator": "userTagProfileLookup", "value": {
    "tagId": {"operator": "in", "value": [<tag1>, <tag2>]},
    "duration": {"operator": "in", "value": ["ALL"]},
    "filter": [{"field": "rank", "operator": "lte", "value": 1000}]}}
  ```

  To choose the tags, find the event (a market's row carries its eventId) and read its tagIds; then rank
  those tags with query_lookup on a tagIds field, filtering id to the event's tagIds and ordering by
  numEvents descending, and keep the first two.

- **Across all markets** — the top 1000 wallets overall:

  ```json
  {"field": "userId", "operator": "userProfileLookup", "value": {
    "duration": {"operator": "in", "value": ["ALL"]},
    "filter": [{"field": "rank", "operator": "lte", "value": 1000}]}}
  ```

The filter goes in `walletFilters` on unifiedSmartMoneySummary, unifiedSmartMoneyHolders and
globalSmartMoney, and in `filter` on signalScore. A saved cohort can be used instead with
`{"field": "userId", "operator": "inCohort", "value": "<cohortId>"}`. Always state which wallet set you
used (the categories, duration and rank cutoff, or the cohort's name).

## Find the event or market

Names are not ids. Resolve an event or market the user names with query_lookup (the `event` or `market`
lookup table through an eventId or marketId field, with `request.searchKey`), and if several results
match, show them and ask. For a Polymarket URL, search on its slug.

## How smart money is positioned on an event

1. **Summary** — `unifiedSmartMoneySummary` with `eventId` (a bare value) and the wallet set in
   walletFilters. It returns one row per open market. Add a marketId filter to get a single market.
   - `smartPct` is smart money's Yes share of its money on the market; compare it with `marketPct`, the
     market's own probability. The gap between them is the story.
   - `yesValue`/`noValue` are dollars still held on each side, `yesWallets`/`noWallets` how many wallets,
     and `yesPrice`/`noPrice` smart money's average entry, not the live price.
2. **Holders** — `unifiedSmartMoneyHolders` with `eventId` and `positionId` (one side of one market),
   ordered by atRisk descending. Query the sibling positionId for the other side. The token lookup
   (query_lookup on positionId) gives each position's name and whether it is the Yes token.
3. **Insider-like bets** — `signalScore` filtered to the eventId and the wallet set, ordered by score.
4. Then ask whether the user wants to compare against another wallet set, and offer concrete options: a
   few relevant existing cohorts from list_cohorts, and sets that fit the event, such as the overall top
   wallets through userProfileLookup, a shorter duration like 1M, or a minimum roi or realizedPnl. Run the
   comparison with the same queries and a different filter.

On a neg-risk event (several mutually exclusive outcomes, like "Who will win?"), the No side of an outcome
is smart money's bets on every other outcome, so read each market's row as "this outcome vs. the field".

## Which markets smart money favors

`globalSmartMoney` ranks open markets by how far smart money's implied price is from the market price.
Pass the wallet set in walletFilters, and to restrict it to categories, `tagIdFilters` with tag ids. Order by
`score` (a 0-100 blend of edge, wallets and dollars at risk) for the strongest overall signal, or by
`yesSlippage` for the biggest mispricing.

- `side` is the outcome smart money favors, `smartMoneyPrice` its implied price for that side and
  `marketPrice` the live one; the edge is their difference.
- Resolve `marketId` to the question with query_lookup on marketId, filtering id with `in`.
- Treat a large edge backed by only one or two wallets or little atRisk as weak.
- A negative `daysLeft` means the end date has passed and the market is waiting to resolve; say so
  rather than presenting it as a trade.

## Insider signals

`signalScore` scores currently held positions in active markets from 0 to 100 for insider-like
conviction. Useful fields: `relSize` (the bet against the wallet's usual size; above 1 is bigger than
usual), `tradeSize` (peak dollars in), `avgEntryPrice` against `pNow`, and `daysToResolution`. Filter it by
eventId, marketId, tagIds or the wallet set. Describe these as signals worth a look, never as proof of
insider trading.

## Presenting results

- Lead with the answer (for example "Smart money is 78% Yes vs. market 62%, $1.4M across 41 wallets"), then
  the table.
- Show prices in cents and money rounded ($1.2M, $45K).
- This is analytics, not financial advice; don't tell the user to place a bet.

## Showing wallets

When your output is rendered as Markdown in a terminal (such as Claude Code, Codex CLI or Gemini CLI),
show every wallet you mention as a Markdown link to its DataDash page:
`[label](https://datadash.xyz/wallet/<address>)`, where `<address>` is the wallet's address (the user id)
and the label is the user's display name if known, otherwise the shortened address. For example, the
wallet `0x56687bf447db6ffa42ffe2204a05edaa20f55839` is shown as
[Theo4](https://datadash.xyz/wallet/0x56687bf447db6ffa42ffe2204a05edaa20f55839) when its display name is
known, and as [0x5668…5839](https://datadash.xyz/wallet/0x56687bf447db6ffa42ffe2204a05edaa20f55839)
otherwise. Holder and signal rows return only the address; resolve display names in one query_lookup on
the userId field, filtering userId with `in` on all the addresses you will show.
