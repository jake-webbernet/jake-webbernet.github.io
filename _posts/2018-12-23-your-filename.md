---
published: false
---
We found that a certain number of queries were running really slow. In this article I want to explore some of the findings I've had when speeding up some of the queries.

## Tables

Below are the tables that are involved in this query

- People - 900,000~ rows
  - This is a table of people.
- Users - 880,000~ rows
  - This is a table of user accounts for people, details such as email address
- Details - 880,000~ rows
  - This lists personal details of users, such as first_name, last_name
- Validation - 393,000~ rows
  - This is a special table that indiciates if a user is validated with a Government registry
- Business People - 870,000~ rows
  - This connects people to businesses
- Businesses - 2000~ rows
  - Businesses can have many people attached to them
  
When designing the data model for this platform I really tried to follow good normalisation practices. That is, trying to split up the responsibilities of the tables as much as possible. I'd always heard though that it was a balance between normalisation and performance, and I believe that one of the reasons we have issues is joining large tables.

## Query

The query in question that was running slow was quite a basic one. A query that allows for businesses to be able to search for a user, fetching data from all of the tables.

```SQL

SELECT people.id,
details.first_name,
details.last_name,
details.dob, users.email, 
validations.valid

FROM people
INNER JOIN users ON people.user_id = users.id
INNER JOIN details ON users.personal_detail_id = details.id
LEFT JOIN validations ON people.validation_id = validations.id
INNER JOIN business_people ON business_people.student_id = people.id
WHERE (business_people.business_id = 1 AND business_people.active is TRUE) AND ((LOWER(details.first_name) = 'smith' OR  LOWER(details.last_name) = 'smith') OR LOWER(users.email) = 'smith') ORDER BY "details"."last_name" ASC LIMIT 20 OFFSET 0 ;

```

So there is a fair bit going on here.

- 3 INNER JOINs
- 1 LEFT JOIN
- 3 OR statements
- 3 LOWERs
- 1 ORDER BY


## Being Honest
In all honesty there was a whole lot more wrong before I implemented some emergency changes to the tables in question

- The query was running originally a LIKE search on the details instead of an exact match - bypassing any and all indexes we previously had on the table
- Some of the indexes were missing, specifically on the `Details` table and the `first_name` and `last_name` columns. This caused huge overhead because not only were we searching directly on the column, we were sorting by it. So a BTREE was added to both.

We got a bit of a speed increase by making these changes but not a whole lot.

## Optimisation Process

* Firstly I had to get copy of the production data locally so we can recreate the same issue and iterate on a solution.

**My goal here is get the time down to less than 10ms**

### Test Results

| Test | Run 1 | Run 2 | Run 3 |
|------|-------|-------|-------|
| Original  | 5506ms | 5214ms | 6082ms |
| 2 - QUERY ~* instead of LOWER | 2484ms | 3911ms | 2677ms |
| 3 - MVIEW / Query with LOWER | 605ms | 489ms | 489ms |
| 3 - MVIEW / Query without LOWER | 399ms | 389ms | 399ms |
| 4 - MVIEW / 3x QUERY UNION ALL   | 6ms  | 10ms | 6ms

**Test 2**
* I thought that using ~* instead of would be a suitable replacement for **LOWER**. 
* My regex is a little rusty and I could see this was giving me wildly different result sets anyway.
* Though still a lot faster than the original, it's not close to where I was aiming for.


**Test 3 -- Materialized View (MVIEW) **
* A [materialized view](https://www.postgresql.org/docs/9.3/rules-materializedviews.html) is neat way to create a **cached** result set of a query
* As with any cache, we have to think about the trade off. This would mean that users were querying a potentially older data set, and new users would not appear for however long we decide to refresh the cache
* I had never tried this out before but figured I would give it a shot
* I created a materialized view (MVIEW) called `user_list` with a modified original query
  * I removed the `WHERE (business_people.business_id = 1)` clause as I would want to pass this into my view later on, and query the list for different businesses.
  * I removed the limit and offset
  * I removed the first_name/last_nam WHERE filters
  
```sql

CREATE MATERIALIZED VIEW user_list
AS

  SELECT people.id,
  details.first_name,
  details.last_name,
  details.dob,
  users.email, 
  validations.valid

  FROM people
  INNER JOIN users ON people.user_id = users.id
  INNER JOIN details ON users.personal_detail_id = details.id
  LEFT JOIN validations ON people.validation_id = validations.id
  INNER JOIN business_people ON business_people.student_id = people.id
  WHERE business_people.active is TRUE

WITH NO DATA;

```
  
* After creating the materialized view, I refreshed the data in it using `REFRESH MATERIALIZED VIEW user_list;`
* I added indexes on all of the columns, as I would be searching on all of them
* I created a unique index as this is required to refresh the materialized view concurrently **(if you don't refresh concurrently the view will lock and no users can select from it)**
* I do have duplicate people_ids in this table as one person can be associated with different businesses, though I used a unique index with both the people_id and business_id

```sql
CREATE INDEX on user_list(first_name);
CREATE INDEX on user_list(last_name);
CREATE INDEX on user_list(email);
CREATE INDEX on user_list(business_id);
CREATE UNIQUE INDEX on user_list(id, business_id);
```

* I tested it out and whilst it took longer `REFRESH MATERIALIZED VIEW CONCURRENTLY user_list;` worked nicely

So with the new MVIEW my query ended up being:

```sql
SELECT * FROM user_list
WHERE (business_id = 1) AND 
(LOWER(first_name) = 'smith' OR (LOWER(last_name) = 'smith') OR (LOWER(email) = 'smith'))
ORDER BY "last_name" ASC 
LIMIT 20 OFFSET 0;

```

***TOTAL TIME ~ 500ms***

Not bad, still pretty slow for a query. I knew that the LOWER was really costing me because we were asking SQL to lower the result set before doing the comparison.

**Test 3**

I moved the LOWER into the view because I figured there is no point in running this expensive function everytime we query.

So i dropped the view and started again - but this time adding a **LOWER** on first_name and last_name

```sql
CREATE MATERIALIZED VIEW user_list
AS

  SELECT people.id,
  lower(details.first_name) as first_name,
  lower(details.last_name) as last_name,
  details.dob,
  users.email, 
  validations.valid

  FROM people
  INNER JOIN users ON people.user_id = users.id
  INNER JOIN details ON users.personal_detail_id = details.id
  LEFT JOIN validations ON people.validation_id = validations.id
  INNER JOIN business_people ON business_people.student_id = people.id
  WHERE business_people.active is TRUE

WITH NO DATA;

```

This means I could remove the LOWER out of my query and end up with

```sql

SELECT * FROM rental_by_category
WHERE (business_id = 1) AND (first_name = 'smith' OR last_name = 'smith' OR email = 'smith')
ORDER BY "last_name" ASC
LIMIT 20 OFFSET 0;

```

***TOTAL TIME ~380ms***


**Test 4**

I had read about `OR` clauses being quite slow, and so I experimented with removing the different OR statements and saw a huge performance boost. How could I achieve an OR type of filter without actually doing one? Enter `UNION ALL`


```sql
EXPLAIN ANALYZE

  SELECT * FROM user_list
  WHERE (business_id = 1) AND (first_name = 'smith')

  UNION ALL

  SELECT * FROM user_list
  WHERE (business_id = 1) AND (last_name = 'smith')

  UNION ALL

  SELECT * FROM user_list
  WHERE (business_id = 1) AND (email = "smith")

ORDER BY "last_name" ASC
LIMIT 20 OFFSET 0;

```

***TOTAL TIME ~ 7ms***

That's within parameters so this is the solution I'm going to go with

**Extra Experimentation**

* Since replacing `OR` with `UNION ALL` made such a difference, I decided to return to my original query that wasn't in an MVIEW, and see if giving it the same treatment would speed it up to an acceptable time. After all, if I can get away with not using an MVIEW, and allow users to query data in real time, I think I would prefer that.
  * **500-600ms** 
   * Is what resulted with the same query, split into 3 parts and removing the `OR` statements, but using `UNION ALL`.
   * But removing the `lower` part of the query which meant the search was not case insesitive
  * **> 3 minutes**
   * Is what resulted when I added lower back in!
   * I also added an index with the `lower`
   
   ```sql
     CREATE INDEX ON details (lower(first_name));
     CREATE INDEX ON details (lower(last_name));
   ```
   
* I played with the concept of making our code only store lower case firstn names and last names, which would mean some type of hook that would need to run everytime someone adds their details, but still, it would be around ~600ms and we would have the added overhead of hidden logic and model hooks which i'm not a fan of.

* The MVIEW seems like the best option, but we have to consider the best way to set it up

## Refreshing the MVIEW

So the MVIEW is cached, thats what makes it so fast.

We need to run `REFRESH MATERIALIZED VIEW CONCURRENTLY user_list;` whenever we want to refresh the data. How often really depends on your needs.

There are ways to do this automatically using `TRIGGERS` however I didn't really want to refresh the MVIEW everytime something changed, I'm ok with doing it every 30 mins for instance.








