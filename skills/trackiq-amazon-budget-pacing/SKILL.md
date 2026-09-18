---
name: trackiq-amazon-budget-pacing
description: Builds a mid-month Amazon advertising pacing report — month-to-date spend against the plan, a day-weighted projection to month end, the gap in dollars, which portfolios are capping out before the day is over, which are underdelivering their share, and a reallocation table that moves budget from the second to the first. Use when the user asks about budget pacing, are we on budget, month-end forecast, will we hit the number, overspending or underspending, budget reallocation, or how much is left to spend this month.
---

# Budget Pacing & Month-End Forecast

Replaces the spreadsheet somebody rebuilds by hand on the 20th. **Are we going
to hit the number, and which budget should move where?**

Output is a branded HTML report: the projection, the portfolios off pace, and
a reallocation table.

## Requires

- The TrackIQ MCP, for `list_marketplaces`, `get_account_overview`,
  `get_campaigns` and `get_portfolios`.
- **The month's plan** — the spend target, and the sales or ACOS target if there
  is one. **This is not in the API.** Ask for it. Without it the report can
  project where spend lands but cannot say whether that is right.
- Nothing else. No filesystem, no shell, no internet.
- **Without the MCP:** works from a daily spend-and-sales export for the month
  to date plus the plan number.

## First run

Fill in a copy of `assets/account.example.md` saved as account.md beside the
skill. Every TrackIQ skill reads the same file, so an account already set up
for another TrackIQ report needs nothing added here.

If the runtime has no filesystem, print the same block and ask the user to
paste it into their project instructions once.

## Read first

- `assets/pulls.md` — the calls, and why you cannot get daily spend per
  campaign in one call
- `assets/method.md` — the day-weighted projection and the reallocation rule
- `assets/checks.md` — what to verify before anything is sent

Copy `assets/report-template.html` and replace every `{{TOKEN}}`.

## Non-negotiables

1. **The plan comes from the user, not the API.** No budget field exists
   anywhere in the MCP — not on campaigns, not on portfolios. If the user cannot
   supply a plan, say the report is a projection without a verdict, and do not
   invent a target from last month's spend without labelling it as that.
2. **You cannot have daily spend per campaign in one call.** `get_campaigns`
   with `granularity="daily"` silently drops the campaign dimension and returns
   an account-level curve. Use the account curve for the projection and campaign
   totals for attribution. Never present campaign totals as if they were a
   trend.
3. **Weight the projection by day of week.** A flat `mtd / days_elapsed x
   days_in_month` is wrong on any account with a weekend pattern. Use the
   observed weekday profile — the method file has the arithmetic.
4. **State the projection as a range, not a number.** One figure implies a
   precision daily spend variance does not support. Give the central case and a
   low/high from the observed daily variation.
5. **`in_budget` is the only budget signal that exists.** `get_portfolios`
   returns `in_budget` as 0/1 — whether the portfolio is currently inside its
   cap. `in_budget = 0` is a portfolio that ran out of money today. It is not a
   budget amount and must never be presented as one.
6. **Underdelivery is not automatically a problem.** A portfolio spending under
   its share at a good ACOS is working. Only call out underdelivery where the
   efficiency would justify more spend.
7. **Reallocation moves money between portfolios, it does not create it.** Every
   proposed increase is funded by a named decrease, and the table sums to zero
   against the plan. Show the sum.
8. **Nothing is changed.** This is a proposal a human applies in the ad console.
9. **Never print `account_id`.**

## The shape of the month

Run it any time, but it earns its keep between day 10 and day 25. Before day 10
the projection is noise; after day 25 there is not enough month left to act.
Say which of those three zones the run falls in.

## What it pairs with

`trackiq-wasted-spend` finds the money already being lost inside the current
budget; this one decides whether the budget itself is the right size and where
it sits. Run the sweeper first — reallocating spend onto keywords that do not
convert just moves the problem.

## Delivery

The output is produced in the chat first. Delivery is the last step and the
method comes from the Delivery block in account.md — never ask per run.

| Method | What to do | Needs |
|---|---|---|
| `in-chat` | Return the report. The default, and the fallback for every other method. | nothing |
| `file` | Write it beside the skill, dated. | a filesystem |
| `slack` | Post the headline findings as text, then upload the file. | a connected Slack tool |
| `n8n` | POST it to the configured webhook. | network access |
| `email` | Hand it to the connected mail tool. | a connected mail tool |

Confirm before the first outward send of a session, fall back to in-chat
loudly when a method is unavailable, and never substitute a different
outward channel.

## Version

`trackiq-amazon-budget-pacing` v1.0.0 (2026-09-18).

If the user asks whether this skill is current, fetch
`https://trackiq.com/skills/registry.json`, compare the `version` field for
`trackiq-amazon-budget-pacing`, and if it is newer, give them the download link and
the one-line changelog. Do not fetch at any other time.
