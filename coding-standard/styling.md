# Styling

```sql
WITH data AS (
  SELECT DISTINCT
    * EXCEPT(field_1, field_2),
    created_at,
    ended_ad,
    modified_by,
    IF(field IS NULL, 0, 500) AS field,
    CASE
      WHEN created_at IS NOT NULL THEN TRUE
      WHEN created_at > '2022-02-01' THEN FALSE
      ELSE NULL
    END AS another_field,
    (amount / 100) * 2 AS number_field,
    status = active AS boolean_field
  FROM database.schema.table short
    LEFT JOIN database.schema.other_table os ON os.date = short.created_at
    LEFT JOIN cte_join USING(modified_by)
    PIVOT(
      SUM(amount) FOR month IN('January', 'February', 'March')
    ) AS p (colum_name, jan, feb, mar)
    LATERAL(TABLE(FIBONACCI_SEQUENCE_UDF(6.0::FLOAT))) 
  WHERE
    created_at > '2022-01-01'
    AND modified_by IS NOT NULL
  GROUP BY
    1, 2
  HAVING
    COUNT(DISTINCT status) > 0
  QUALIFY
    ROW_NUMBER() OVER(PARITION BY modified_by) = 1
  ORDER BY 
    1 DESC,
    2 ASC
  LIMIT 100
)

SELECT
  created_at,
  ended_at,
  modified_by,
  field,
  another_field,
  number_field,
  boolean_field
FROM data
```

Syntax option availability may differ from dialect to dialect, and their versions.

### Main keywords after which references go to new line with indentation

```sql
SELECT
  <fields>
CASE
  <when|else>
FROM <table>
  <join>
PIVOT
  <statement>
WHERE
  <condition>
GROUP BY
  <field>
HAVING
  <condition>
ORDER BY
  <field>
```

### Keywords that require "completion" in the same line

```sql
FROM <table>
LEFT JOIN <table> <alias> ON <field> = <field>
INNER JOIN <table> USING(<field>)
PIVOT(AGG(<field>) FOR <field> IN (<value>, <value>)) <alias>
LATERAL(TABLE())
```

### Exceptions

#### Simple SELECT

If it's a simple `SELECT *` query that is used for multiple `UNION`s, you can write it in one line, if it doesn't require you to list all columns

```sql
SELECT * FROM table_a UNION ALL
SELECT * FROM table_b
```

#### Keywords followed by number

If a keyword is followed by only one number or field reference, we can put it on the same line

```sql
GROUP BY 1
ORDER BY 2
LIMIT 100
```

But if we have multiple fields, using indentation only from the second item can decrease readability

```sql
GROUP BY field, 
  field2,
  field3
```

#### Complicated JOINs

If a JOIN has many conditions, putting them all in one line can lead to decrease in readability, therefore we can indent the conditions, starting from the second.

```sql
FROM table
  LEFT JOIN other_table t ON t.user_id = table.user_id
                             AND t.date = table.date
                             AND t.country = table.country
                             AND t.currency = table.other_currency
```
