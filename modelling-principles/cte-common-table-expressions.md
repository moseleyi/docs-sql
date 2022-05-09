# CTE - Common Table Expressions

### Use Common Table Expressions <a href="#use-common-table-expressions" id="use-common-table-expressions"></a>

### Split your code <a href="#split-your-code" id="split-your-code"></a>

Break your complicated queries and JOINs into reusable and readable chunks of code using CTEs. Use it when:

* you need to deduplicate records
* you need to use window functions like: `ROW_NUMBER()`, `LAG()`, `LEAD()`
* a simple join is not enough and you need to apply sorting, `WHERE` or `HAVING` conditions before the join can happen

### Syntax

Make the name of the CTE understandable but not unnecessarily long. It needs to indicate what you're doing with the code in this block or what kind of data source it is:

* transformation - `add_totals_in_usd`
* transformation - `remove_duplicates`
* source - `payment_ledger`
* source - `subscription_audit`

Keep keyword `AS` and opening parenthesis on the same line.&#x20;

Indent the code in CTE to give visual indication it "belongs" to it.

After closing parenthesis, keep the comma in the same line, and separate CTEs with a clear line.

```sql
WITH <name> AS (
),

<name> AS (
)
```

### Final CTE <a href="#final-cte" id="final-cte"></a>

Wrap your latest code in a CTE called `final` and then `SELECT` from it by listing all columns. That allows people who are not familiar with the model to look quickly what fields are being returned, without the need to query the table.

```sql
WITH  data AS (
  SELECT
    subscription_id,
    LOWER(status)                                                           AS status,
    CONVERT_TIME_TO_DATE_TZ(action_at)                                      AS from_date,
    LEAD(hist_date) OVER(PARTITION BY subscription_id ORDER BY hist_date)   AS next_date,
  FROM transformations.dim_subscriptions_audit
),

final AS (
  SELECT
    calendar.date,
    subscription_id,
    COALESCE(LAG(status) OVER(PARTITION BY subscription_id ORDER BY calendar.date), 'new') AS previous_status,
    status,
    COALESCE(status <> previous_status, TRUE)                               AS status_changed,
    LEAD(status) OVER(PARTITION BY subscription_id ORDER BY calendar.date)  AS next_status,
    from_date,
    until_date
  FROM data
    LEFT JOIN transformations.dim_calendar calendar ON calendar.date BETWEEN from_date AND until_date
)

SELECT
  date,
  subscription_id,
  previous_status,
  status,
  status_changed,
  next_status,
  from_date,
  until_date
FROM final
```

\
