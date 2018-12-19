* Get comfortable with profiling queries using the `EXPLAIN` and `ANALYZE`. Be careful with Analyze as it will actually run the query to give you more information such as memory used, and time taken.
  * Look out for `Seq Scans` as they indicate that the whole table is being scanned
    * Not a huge deal when the table is small, but on large tables this is a show stopper and can cause crashes.
  * Sometimes the database will opt for a `Seq Scan` instead of an index scan depending on what it thinks is more efficient
  * Yes, even your active record queries need to be profiled. Add `.explain` to the end of the query and print this to console

## Remember ##
* Always ensure columns that are used with an `ORDER BY` or a `WHERE` clause are indexed
* Using the `LIKE` operator automatically triggers a full table scan (Seq Scan) even if you are using a BTREE index on said column
* `OR` statements accross tables that you have joined trigger a full table scan and are quite expensive

