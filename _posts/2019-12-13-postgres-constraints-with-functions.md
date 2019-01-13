---
published: true
tags: sql
---

Often times I've wondered just how powerful the Postgres `CHECK` feature is. Not often delving into the database and living mostly in Rails, I feel that I have only scratched the surface of its capabilities.

## Problem

I've got two tables - `sessions` and `days`. Each session can have multiple days, though it's very common that I need to show the first day that a session starts. I've tossed and turned on whether to just cache that value, but I really didn't want the headache of having to constantly pop the cache. I also needed a high amount of accuracy here, and I didn't think a cache would offer that as it's absolutely imperitive that the session start date is it's correct value.

Instead, I opted to go against normalisation and add a `start_datetime` field to the `sessions` table. My plan was keep this column consistently updated with the earliest date and time found in the corresponsing `days` table for the session. This would make it very easy in the application to fetch and query the first date without having to to run costly comparisons every time. 


# Solution

As I said, I have a pretty limited understanding on the `CHECK` feature, but I thought that it could come in handy in this scenario.

Firstly, I wanted to add a `CHECK CONSTRAINT` to the `sessions` table so that the new `start_datetime` column could not possibly reflect an incorrect date based on the the entries in the `days` table.

I needed to create a function that would do a few things:

* Accept 3 parameters, the `start_datetime`, the `end_datetime`, and the `session_id`
* Scan over the `days` table and ensure that the new `start_datetime` indeed matched the earliest `start_datetime` in the days table
* Scan over the `days` table and ensure that the new `end_datetime` indeed matched the latest `end_datetime` in the days table

Here is what I came up with:

```sql
CREATE OR REPLACE FUNCTION session_date_range(_start_datetime INTEGER, _end_datetime INTEGER, _session_id INTEGER)
RETURNS BOOLEAN AS
$$
BEGIN
IF
_start_datetime IS NULL AND _end_datetime IS NULL
THEN return TRUE;
END IF;

IF
_start_datetime = (
                      SELECT start_datetime
                      FROM session_dates
                      WHERE session_dates.session_id = _session_id
                      ORDER BY start_datetime DESC LIMIT 1
                      ) AND

_end_datetime = (
                      SELECT end_datetime
                      FROM session_dates
                      WHERE session_dates.session_id = _session_id
                      ORDER BY end_datetime DESC LIMIT 1
                      )
THEN RETURN true;
ELSE RETURN false;
END IF;
END;
$$ LANGUAGE PLpgSQL;
```

I could test this out with:

```
SELECT session_date_range(111,222,1);
# => f
```

A few things I learned while building my first function:
* It's a good idea to prefix arguement variables passed into the function with an underscore `_my_argument_variable`. This is so it doesn't clash with an actual column name in the function
* Functions that we intend to use in a `CHECK` need to return a boolean data type
* I needed to add a `NULL` check at the start of the script otherwise when I update the `sessions` table and make no changes to the time, it would error. I think this might be a bit of a hack and might be due to something I'm not doing in my comparison

The function seemed to work great, now it was time to add it to the table and see if it works

```
ALTER TABLE sessions ADD CONSTRAINT session_date_range_const
CHECK (session_date_range(start_datetime, end_datetime, id));
```

This is adding the constraint to the existing table, and passing the `start_datetime` and `end_datetime` as well as the session ID in question.

```sql

SELECT * FROM session_dates WHERE session_id = 11;

   id   | session_id | start_datetime | end_datetime |
--------+------------+----------------+--------------+
 194303 |         11 |     1544306400 |   1544320800 |
(1 row)

-- try and update a session start time to be earlier than the associated days
UPDATE sessions SET start_datetime = 1, end_datetime = 2 WHERE id = 11;
ERROR:  new row for relation "sessions" violates check constraint "session_date_range_const"
DETAIL:  Failing row contains (11, 11, 2018-12-08 08:11:33.756823, 2018-12-08 08:11:33.756823, 101574, 0, 1, 2, 0, t, , 2, 60.0, null, 2, 3, c07f680f59fa23a75911, 1).

-- update to the same days that are present in the days table
UPDATE sessions SET start_datetime = 1544306400, end_datetime = 1544320800 WHERE id = 11;
UPDATE 1

```

### Testing

Even though this was a function that lived on the database, I still wanted to test it from my Ruby on Rails application.

I opted for a model spec in Rails, just testing that I was prevented from updating the column to the incorrect value.


I ended up adding this [Great custom matcher](http://www.techienov.com/snippet/rspec/custom-check-constraint/) in RSPEC so that I could accurately test that my new constraint was working correctly in the spec below.

```
require "rails_helper"

RSpec.describe Session do
  subject { described_class }

  let(:session) { create(:session, start_datetime: nil, end_datetime: nil) }

  before do
    SessionDate.create(start_datetime: 1, end_datetime: 2, session_id: session.id) 
  end

  describe 'database constraints' do
    describe 'session_date_range' do

      context "when the session start_datetime does match the first session_date start_time" do
        it "allows me to update" do
          expect { session.update(start_datetime: 1, end_datetime: 2) }.to_not raise_error
        end
      end

      context "when the session start_datetime does NOT match the first session_date start_time" do
        it "causes an error" do
          expect { session.update(start_datetime: 10, end_datetime: 2) }.to violate_check_constraint(:session_date_range)
        end
      end

      context "when the session start_datetime or end_datetime is nil" do
        it "does not cause an error" do
          expect { session.update(start_datetime: nil, end_datetime: nil) }.to_not violate_check_constraint(:session_date_range)
        end
      end
    end
  end
end
```

So that's the validation taken care of, how am I going to keep the field actually up to date?

## Trigger

I wanted to try my hand at my first Postgres trigger. Database triggers seem pretty contentious in the development community, mostly around the fact that they hide logic that can be easily missed. I don't disagree with this premise though for this use case I thought it would be a good chance to try it out and make my own decision.

So the idea was to create a trigger that would run on the `days` table that would automatically update the parent `session` start and end times based on the same logic implemented in the check.

Here is what I came up with:

```sql
CREATE OR REPLACE FUNCTION refresh_parent_session_time()
RETURNS TRIGGER
SET SCHEMA 'public'
LANGUAGE plpgsql
AS $$
DECLARE
earliest_start_datetime INTEGER;
latest_end_datetime INTEGER;
BEGIN
earliest_start_datetime := (
                            SELECT start_datetime
                            FROM session_dates
                            WHERE session_dates.session_id = NEW.session_id
                            ORDER BY start_datetime ASC LIMIT 1
                            );


latest_end_datetime := (
                        SELECT end_datetime
                        FROM session_dates
                        WHERE session_dates.session_id = NEW.session_id
                        ORDER BY end_datetime DESC LIMIT 1
                        );

UPDATE sessions
SET start_datetime = earliest_start_datetime, end_datetime = latest_end_datetime
WHERE id = NEW.session_id;

RETURN NEW;
END;
$$;

```

And I learned: 
* I was able to declare local variables that can be passed throughout the function via the `DECLARE` block
* I can use the `:=` syntax to assign values to my local variables
* I have access to a list of [special variables](https://www.postgresql.org/docs/9.1/plpgsql-trigger.html) such as `NEW` when i'm working with a trigger. `NEW` is the whole record that is being added, and essentially what is 'triggering' the 'trigger'


Now to attach the trigger to the table:


```sql

-- easiest way - will fire the trigger anytime the table is inserted, updated or deleted on
CREATE TRIGGER trig_refresh_session_time
    AFTER INSERT OR UPDATE OR DELETE ON session_dates
    FOR EACH ROW EXECUTE PROCEDURE refresh_parent_session_time();
    
-- can also pass a 'when' boolean expression so that it's only fired when the values of start and end time are changed!
CREATE TRIGGER trig_refresh_session_time
    AFTER UPDATE ON session_dates
FOR EACH ROW
WHEN (OLD.start_datetime IS DISTINCT FROM NEW.start_datetime
   OR OLD.end_datetime IS DISTINCT FROM NEW.end_datetime)
EXECUTE PROCEDURE refresh_parent_session_time();


-- I used the first one because the dates time only has three fields.
CREATE TRIGGER
```

And lets test it out:


```sql

-- checkout the current sessions times

SELECT * FROM sessions WHERE id = 11;

-[ RECORD 1 ]----+---------------------------
id               | 11
start_datetime   | 1544306400
end_datetime     | 1544320800

-- yep, they match the session_dates

SELECT * FROM session_dates WHERE session_id = 11;

   id   | session_id | start_datetime | end_datetime |
--------+------------+----------------+--------------+
 194303 |         11 |     1544306400 |   1544320800 |
(1 row)


-- Update the session_dates and ensure the parent session is updated

UPDATE session_dates SET start_datetime = 11223344 WHERE id = 194303;
UPDATE 1

SELECT * FROM sessions WHERE id = 11;
-[ RECORD 1 ]----+---------------------------
id               | 11
start_datetime   | 11223344
end_datetime     | 1544320800
```

All working correctly.

I think this is a pretty good use case for triggers as the code will very rarely change - my only concern would be other developers perhaps getting confused in testing when dates are updated automatically.

There is definately logic that can be DRYed up as these functions essentially use the same ORDER BY query to fetch the earliest and latest dates. It would be best to make this into it's own Postgres function and reference them in the views.

I think I will use this method a lot for constraint work, though I will probably hold off on using triggers for anything more complex as I still think it could present issues in a team setting. The last place I would be looking when experiencing a bug locally is the Database Triggers. Maybe not any more. 
