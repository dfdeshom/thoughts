Upserting rows in postgresql 9.5 with Spark
===============================================


Although the current Postgres JDBC data source allows SELECT and INSERT operations with Spark, it doesn't allow for upserts. Since Postgres 9.5 now finally support his handy operation, let's see how we can fake support of it through psycopg2. First, some background on some pitfalls of implmenting upserts

## Pie in the sky

Part of the reason Spark only supports INSERTs is that SQL database support for upsert operations varies a lot; up until recently Postgres did not even support it. The other reason is that upserts are not generally efficient at scale no matter how you approach them. There are 2 main approaches:

- Server-side: let the database worry about data integrity and enforcing uniqueness constraints. If your constraint is expensive or you have hundreds of clients upserting, this operation will be slow: checking the constraint does not parallelize very well and requires a full  scan of the index created on the uniqueness constraint in some cases. You can see a great critique fo this approach here: https://github.com/apache/spark/pull/16692#issuecomment-274977343 . On some databases like Postgres, upserts require a UNIQUE index on the key or constraint to be upserted. That's just another operational detail to worry about.

- Client-side: let the client/code worry about data integrity. In this case, the client has to worry about identifying uniqueness constraints and merging records together. This can be quite error-prone. In addition, this approach means that  your database cannot guarantee the integrtiy of your data, any inconsistencies have to be resolved at the code level. You can see a good critique of this chalenges of this approach here: 
https://github.com/apache/spark/pull/16685 .

There are drawbacks everywhere, but for our case we chose the server-side aproach. The main reasons were:
- Using postgres to guarantee data ingrity was a must
- our constraints would be simple fields, which would not be too expensive for the  DB to check against

## Generating records

The easiest way to imagine how an upsert operation would take place would be to have an array of dicts, with the following constraints:

* all dicts in the array have the same fields

* the constraining field is present on all dicts

* the constraining field's values are unique. This is to avoid 2 or more workers updating the same record in the database at the same time. This will lead to conflicts and the upsert operation will be canceled.


## Generating the upsert statement programatically

The upsert statement itself is pretty straightforward. It's similar to a INSERT statement, with the the addition of a `ON CONFLICT DO UPDATE SET` clause. Here is what a statement might look like for upserting values into a table where the uniquesness constraint is on the `ID` field: 

```sql
INSERT INTO TABLE T(ID, ATTR1, ATTR2) VALUES (1,'A', 'B')
ON CONFLICT(ID) DO UPDATE SET
ATTR1=EXCLUDED.ATTR1,
ATTR2=EXCLUDED.ATTR2;
```

Every upsert statement looks like this so it's pretty easy to generate them programatically. One needs the table name, columns to update and the ID field. Here is a function that returns a statement given those parameters:  
https://gist.github.com/dfdeshom/89497f7dcd81ad05464b19545a0094e2#file-upsert_statement-py-L1-L23

## Doing the upsert

After generating upsert statement for each item in the RDD, we just need to execute them. In reality we only need to generate the statement for one item: the handy function `psycopg2.extras.execute_batch` will execute the statement in batch against all dict entries in the RDD  : https://gist.github.com/dfdeshom/89497f7dcd81ad05464b19545a0094e2#file-upsert_rdd-py . We're using `mapPartition` here so we don't instantiate thousands of PG connections. We're also using `collect()` to force the RDD to execute all the upsert statements we generated
