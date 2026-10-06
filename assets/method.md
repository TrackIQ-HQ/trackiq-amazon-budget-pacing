# Method

## The day-weighted projection

A flat run rate —

```
projection = mtd_spend / days_elapsed x days_in_month        # DON'T
```

— is wrong on any account with a weekday pattern, and every Amazon account has
one. Run it on a month that has seen four weekends and two, and it swings by
several percent in either direction for no real reason.

Weight by day of week instead:

```
for each weekday w in Mon..Sun:
    mean_spend[w] = mean(total_spend on days of weekday w so far this month)

# fall back to the overall daily mean for any weekday not yet seen
projection = mtd_spend + sum(mean_spend[weekday(d)] for d in remaining days)
```

Ten days in, some weekdays have one observation and some have two. That is
fine — it is still closer than pretending Saturday spends like Tuesday. Say in
the method note how many observations each weekday had if the month is young.

## The range

```
sigma        = stdev(daily total_spend so far)
remaining    = number of days left in the month
low          = projection - sigma x sqrt(remaining)
high         = projection + sigma x sqrt(remaining)
```

Report the central case with the low and high beside it. One number implies a
precision that daily spend variance does not support, and a client who is
handed a single figure will hold you to it.

## The verdict

```
gap        = projection - plan
gap_pct    = gap / plan
daily_need = (plan - mtd_spend) / days_remaining     # the rate needed from here
```

| Verdict | Test |
|---|---|
| **On pace** | `abs(gap_pct) <= 3%` |
| **Over** | `gap_pct > 3%` |
| **Under** | `gap_pct < -3%` |

Three percent is a working default, not a law. If the client has a tighter
tolerance, use theirs and say which was used.

Beside the verdict, give `daily_need` against the current daily mean. "You have
been spending $4,900 a day and need to average $3,780 for the rest of the
month" is the sentence people act on — more so than the projection itself.

## Which zone the month is in

| Days elapsed | What to say |
|---|---|
| **1–9** | The projection is noise. Show it, label it unreliable, lead on the daily rate instead. |
| **10–25** | The useful window. Full report. |
| **26+** | Too late to change the month. Lead on what to set up for next month. |

State the zone. A client reading a day-6 projection as gospel is a worse
outcome than not running the report.

## Efficiency, not just volume

Spend pacing on its own can be met by buying worse traffic. If the client gave
a sales or ACOS target, carry it in parallel:

```
acos_mtd       = total_spend / attributed_sales
acos_projected = acos_mtd                 # assume mix holds; say so
```

Flag the case where spend is on pace and ACOS is drifting — that is the failure
the pacing number hides, and it is worth more than the pacing number itself.

## Off-pace portfolios

Two lists, from portfolio totals:

**Capping out.** `in_budget = 0`. The portfolio stopped serving today. Rank by
spend — a big portfolio hitting its cap is losing more demand than a small one.

**Underdelivering.** Spend share below its share of the same period last month,
**and** ACOS at or better than the account average. The efficiency condition is
what makes it worth funding. A portfolio spending less at a bad ACOS is not
underdelivering, it is being sensible.

Do not list a portfolio as underdelivering on volume alone.

## The reallocation table

Every increase is funded by a named decrease.

```
from:  portfolio, current spend, ACOS, reason      (worst ACOS first)
to:    portfolio, current spend, ACOS, headroom    (best ACOS first)
sum of moves = 0
```

Show the sum. A reallocation table that does not balance is a budget increase
wearing a disguise, and the client will find out at the end of the month.

Cap any single move at 20% of the source portfolio's month-to-date spend.
Bigger moves than that destabilise delivery and the campaigns will not spend it
in the days remaining anyway.

## What this skill does not do

- **No budget changes.** Nothing is written to the account. The output is a
  table someone applies in the console.
- **No forecast of sales.** It projects spend, which is controllable. Projecting
  sales from seventeen days of a month invites an argument the report will lose.
- **No campaign-level daily trend.** The API will not give it without one call
  per campaign. See `assets/pulls.md`.
