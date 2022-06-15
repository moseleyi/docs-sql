# Values like Statuses, Categories

There are two aspects of string values like statuses, categories, reasons, or other descriptive columns that are repeated in a table.

It's advised to keep values that are used for programmatic manipulations as snakecase. For example:

`status = 'fully_active` instead `status = 'Fully Active`

There are few reasons for it:

* Spaces and other characters can cause problems in SQL. Pivot columns can be dynamically created from values that have spaces or start with a number
* Using spaces, hyphens, and other characters increases the chances people may forget what is the value they want. When an analyst writes a query it's much more productive to be able to know, based on such standard, what the value will be. It's especially important for case-sensitive searches.

However, this value also has a purpose of being presented to the user via BI Tool. In order to present it in a best human-readable form, we can change its appearance at the last point of the data manipulation, that is either a business intelligence platform or a label at the end of the query.

You can create a simple function that capitalises words and then is used in your BI Tool like Looker.&#x20;

`sql: lookerise_string(${TABLE}.status}) ;;`

Would have the following impact:

In SQL Table: `fully_active`

In Looker: `Fully Active`

Which looks nicer and these human-readable values would be used for filter suggestions as well but it wouldn't impact the underlaying value which are machine-readable.&#x20;
