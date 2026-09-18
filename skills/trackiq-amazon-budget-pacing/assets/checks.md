# Before you send it

## 1. The plan

- The plan figure is stated on the report, with where it came from.
- If no plan was supplied, the report says it is a projection without a verdict
  — and there is no verdict anywhere on the page.
- A plan inferred from last month is labelled as inferred, every time it appears.

## 2. The window

- The range ends **yesterday**, not today. A partial day drags the run rate down.
- Days elapsed and days remaining are stated and add up to the month.
- The zone (1–9 noise / 10–25 useful / 26+ too late) is named.

## 3. The projection

- Day-weighted, not flat. Spot-check one weekday: does its mean match the days
  of that weekday in the source data?
- A low and a high are shown beside the central case.
- If a weekday has only one observation, the report says the month is young.
- The projection is never presented to the dollar.

## 4. The attribution

- Campaign and portfolio figures are **totals for the period**, and are not
  drawn as a trend line anywhere.
- Campaign spend sums to something close to account `total_spend` for the same
  range. A large gap means campaigns were filtered or paginated short — find out
  which before sending.
- `in_budget = 0` portfolios are described as having hit a cap, never as having
  a budget of a stated size.
- If every portfolio reads `in_budget = 1`, the report says caps were checked
  and none were hit.

## 5. The reallocation

- **The moves sum to zero.** Print the sum on the page.
- Every increase names the decrease funding it.
- No single move exceeds 20% of the source portfolio's month-to-date spend.
- Nothing is proposed into a portfolio whose ACOS is worse than the account
  average.
- Underdelivering portfolios passed the efficiency test, not just the volume
  test.

## 6. Sanity

- If spend is on pace but ACOS has drifted, that is the headline — not the
  pacing verdict.
- Month-to-date spend matches the sum of the daily curve. If it does not, the
  curve has a gap and the projection is built on a hole.
- Percentages against the plan are not quoted to two decimals.

## 7. Render check

```js
({ overflows: document.documentElement.scrollWidth > document.documentElement.clientWidth,
   tables: document.querySelectorAll('table').length,
   rows: [...document.querySelectorAll('table')].map(t => t.querySelectorAll('tbody tr').length),
   logos: [...document.images].map(i => i.naturalWidth > 0),
   // the reallocation table is the last one; its moves must sum to zero
   moves: [...document.querySelectorAll('table')].slice(-1)[0]
            .querySelectorAll('tbody tr').length })
```

`overflows` false and `logos` all true. Then look at it; if it will not paint,
say the check was structural.

## 8. Ship

Save as `<client>-budget-pacing-<YYYY-MM-DD>.html`. Dated by run day — a pacing
report is worthless without knowing which day of the month it was run on.

Lead the message to the client with the daily rate needed from here, not the
projection. That is the number they can act on today.
