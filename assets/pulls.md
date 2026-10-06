# The pull sequence

## 0. Account and the plan

`list_marketplaces` first — several TrackIQ MCPs can be connected with
identical tool names. Never print `account_id`.

Then **ask for the plan** before pulling anything:

| Input | Needed | Notes |
|---|---|---|
| Monthly spend target | yes | the number the report is judged against |
| Sales or ACOS target | optional | turns "on pace" into "on pace and efficient" |
| Portfolio-level splits | optional | if the client budgets by portfolio |

There is **no budget field anywhere in the MCP.** Not on campaigns, not on
portfolios, not on the account. If the user does not have a plan, say so on the
report: it becomes a projection without a verdict.

## 1. The daily curve — this is the backbone

```
get_account_overview(account_id, start_date=<1st of month>, end_date=<yesterday>)
```

Returns one row per day:

`total_revenue`, `attributed_sales`, `organic_sales`, `total_spend`, and the
split by ad type — `sp_spend`/`sp_sales`, `sb_`, `sbv_`, `sd_`.

`total_revenue = attributed_sales + organic_sales` on every row checked, so the
organic share is a real number and worth showing beside the spend.

**End the range at yesterday.** Today is partial and will drag the run rate
down. If the user insists on including today, label the day partial and exclude
it from the projection.

Pull the **same span of the previous month** as well, for a like-for-like
comparison at the same day number.

## 2. Campaign and portfolio totals

```
get_campaigns(account_id, start_date, end_date, limit=200)       # granularity total
get_portfolios(account_id, start_date, end_date, limit=100)
```

Campaign rows: `campaign_id`, `name`, `state`, `portfolio_id`, `ad_type`,
spend, sales, orders, units, acos, roas, cpc, ctr, cvr.

Portfolio rows: `portfolio_id`, `name`, `state`, **`in_budget`**, spend, sales,
acos, roas.

## 3. The trap: daily and per-campaign are mutually exclusive

```
get_campaigns(..., granularity="daily")
```

does **not** give you daily spend per campaign. The campaign dimension
disappears entirely — rows come back as `{date, impressions, clicks, spend,
sales, orders, units, acos, roas, cpc, ctr, cvr}` with no `campaign_id` and no
`name`. It is an account-level curve, the same one `get_account_overview`
gives you with more columns.

So:

- **Projection** → the account daily curve
- **Attribution** (who is spending it) → campaign and portfolio totals for the
  period

Per-campaign daily would be one call per campaign. Do not do that for a
hundred campaigns; it is not worth the round trips for a pacing report. If the
user needs the daily curve for one specific campaign, pull that one.

## 4. `in_budget` is the only budget signal in the product

`get_portfolios` returns `in_budget` as 0 or 1: whether the portfolio is inside
its cap right now. A portfolio at `in_budget = 0` stopped serving today because
it ran out of money — that is exactly the "capping out before day end" signal
the report needs.

It is **not** a budget amount. It does not say what the cap is, how early it
was hit, or how much demand went unserved. Report it as a flag and say what it
means.

If every portfolio reads 1, say the account showed no caps hit rather than
implying nothing was checked.

## 5. Optional

- `get_campaign_groups` — if the client organises above portfolio level
- `get_product_categories_performance` — **two overlapping taxonomies come back
  in one response** (Pacvue-native categories and TrackIQ `(tiq)` tags return
  identical figures for the same campaigns). Pick one and say which. Summing
  everything double-counts spend.
