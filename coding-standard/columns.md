# Columns

### Always lowercase

References to man-made entities: coluns, aliases, names of CTEs, tables, schemas, are to be written in lowercase.

`AS status` not `AS STATUS`

`FROM database.schema.table_name`

`JOIN cte_name AS name`

`WITH cte_name AS ()`

:exclamation:_Snowflake and its drivers DISPLAY everything in uppercase but it doesn't mean it lives as uppercase in the underlaying structure. If you create a column as uppercase, you will still be able to  use the lowercase reference, however in `information_schema`, it will show up as uppercase, which may have an impact on queries to the tables in that schema._ :exclamation:

### Always snake\_case

Column names that contain more than one word are to be written in `snake_case`

* `subscription_id` not `SubscriptionId`
* `order_created_at` not `OrderCreatedAt`

### Trailing comma after last column

Some dialects do not allow leaving comma after last column, for example Snowflake. Only apply this rule if dialect allows it.

For example, Snowflake will return the following error:

{% hint style="warning" %}
\[42000]\[1003] SQL compilation error: syntax error line 40 at position 0 unexpected 'FROM'.
{% endhint %}

### Align aliases when possible

It’s clearer to scan the code by a column of interest, if they’re aligned

```sql
SELECT
  subscription_id,
  LOWER(status)                                                           AS status,
  CONVERT_TIME_TO_DATE_TZ(action_at)                                      AS hist_date,
  LEAD(hist_date) OVER(PARTITION BY subscription_id ORDER BY hist_date)   AS next_date,
  LEAD(status) OVER(PARTITION BY subscription_id ORDER BY hist_date)      AS next_status,
  LAG(status) OVER(PARTITION BY subscription_id ORDER BY hist_date)       AS prev_status,
  hist_date                                                               AS from_date,
  COALESCE(DATEADD(DAY, -1, next_date), CURRENT_DATE())                   AS until_date
```

Instead of:

```sql
SELECT
  subscription_id,
  LOWER(status) AS status,
  CONVERT_TIME_TO_DATE_TZ(action_at) AS hist_date,
  LEAD(hist_date) OVER(PARTITION BY subscription_id ORDER BY hist_date) AS next_date,
  LEAD(status) OVER(PARTITION BY subscription_id ORDER BY hist_date) AS next_status,
  LAG(status) OVER(PARTITION BY subscription_id ORDER BY hist_date) AS prev_status,
  hist_date AS from_date,
  COALESCE(DATEADD(DAY, -1, next_date), CURRENT_DATE()) AS until_date
```

:exclamation:If there’s an exceptionally long column, do not align all aliases to the longest one, as that could decrease the readability. Instead leave the long one on its own but align other ones in a best possible way.

```sql
SELECT
  started_at      AS created_at,
  status          AS entity_status,
  COALESCE(LAG(status) OVER(PARTITION BY subscription_id ORDER BY calendar.date), 'new') AS previous_status,
```

### Use previous fields' references in derivative columns

If dialect allows, use the newly created columns in derivative fields to decrease the length of code

```sql
SELECT
  amount,
  amount / fx_rates.rate AS amount_usd,
  amount_usd / total     AS amount_pct
```

Unfortunately some dialects won't allow us to do it and we have to either use the whole definition or, if there's a lot of code, move it to separate CTE for clarity

```sql
SELECT
  amount,
  amount / fx_rates.rate           AS amount_usd,
  (amount / fx_rates.rate) / total AS amount_pct
```

### No back ticks for aliases

If the name of the column could equal to one of the keywords like `from`, `range`, or `between`, then search for a better name of the column.

```sql
SELECT
  LOWER(status) AS status
```

Instead of

```sql
SELECT
  LOWER(status) AS `status`
```
