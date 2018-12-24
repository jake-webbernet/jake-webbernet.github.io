---
published: true
tags: sql
---

I thought CTEs were basically a neater way to use a subquery with SQL.

From a performance standpoint in Postgres, they have performance differences.

The original challenge I was facing was that I had the below query needing to use common filters, such as the business ID. This was going to be a pain because I wanted to add a series of overall filters such as `business_id` to the whole query. In this first attempt, the most obvious solution was to add the filter to both queries. I didn't think this was going to scale well.

```sql
SELECT  general_people.* FROM general_people
WHERE lower_first_name = 'john AND business_id = 1

UNION

SELECT general_people.* FROM general_people
WHERE lower_last_name = 'smith' AND business_id = 1

LIMIT 20;
```

Searching online I found that you could add filters with a simple CTE. Moving the `business_id` outside the CTE. In all honesty I probably could have lived with this solution, but there was inefficiency that each subquery was returning all rows AND THEN filtering further out rows without a `business_id` of 1.


```sql
WITH CTE AS (
 SELECT  * FROM general_people
 WHERE lower_first_name = 'john
 
 UNION
 
 SELECT * FROM general_people
 WHERE lower_last_name = 'smith'
)

SELECT * FROM CTE WHERE business_id = 1
LIMIT 20;
```

I thought Postgres would have been smart enough to move the filter up into the CTE, but it turns out that by design CTEs are an [optimization fence](https://robots.thoughtbot.com/advanced-postgres-performance-tips#common-table-expressions-and-subqueries) and deliberately prevent filters bubbling up to the CTE.

After some further research I found that normal subqueries seemed to do what I'm looking for!

```sql
SELECT * FROM (
 SELECT * FROM general_people
 WHERE lower_first_name = john
 
 UNION
 
 SELECT * FROM general_people
 WHERE lower_last_name = 'smith'
) test WHERE business_id = 1
LIMIT 20;
```

After running the query planner I could see that both subquery was also filtering on the business.
