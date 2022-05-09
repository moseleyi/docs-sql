# Comments

Add inline comments in sql using `--`if a specific field is not self-explanatory or if you need to apply conditions to it that are not known.

```sql
  WHERE
    -- Deprecating page views that were implemented as tracks
    event_name NOT LIKE '%_viewed'
```

Use `/** */` block comments to add more extensive description of the whole model or CTE. Try to break up the lines so that it doesn’t wrap automatically

```sql
/**
  On 14th March 2022, a change was made in Production db that affected several subscriptions to be incorrectly marked as INACTIVE.
  In order to preserve data integrity, we won't delete those audit records, hence need to exclude them from the Data Model
  We used revision_id for simplicity but here are the details:
  subscription_id IN(140301,112304,219951,213401,213001,223501,208552,62152,131653,259551,230453,225552,219701,47453,75951,122802,140402,140403,140551,154001,158704,170651,240951,241101,145451,129701,165209,84952,145554,62201,206101,105251,131654,201601,179701,119801,198354,181003,183852,184302, 119252)
  AND last_modified_date BETWEEN '2022-03-14 10:34:27.365016' AND '2022-03-14 10:35:05.480551'
 */
WITH base AS (
  SELECT
    id AS subscription_id,
    status,
    last_modified_date
  FROM prod_db.audit_subscription
  WHERE
    revision_id NOT IN(116580,116568,116577,116603,116587,116597,116606,116583,116582,116598,116566,116575,116576,116590,116573,116571,116605,116595,116586,116589,116592,116572,116599,116567,116569,116585,116591,116584,116600,116578,116604,116581,116593,116570,116574,116601,116579,116594,116602,116588,116596)
)
```
