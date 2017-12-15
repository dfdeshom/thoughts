Upserting rows in postgresql 9.5 through spark
===============================================


Although the current Postgres JDBC data source allows SELECT and INSERT operations with Spark, it doesn't allow for upserts. Since Postgres 9.5 now finally support his handy operation, let's see how we can fake support of it through psycopg2. First, some background on some pitfalls of implmenting upserts

Pie in the sky
----------------
Part of the reason Spark only supports INSERTs is that SQL database support for upsert operations varies a lot; up until recently Postgres did not even support it. The other reason is that upserts are not generally efficient at scale no matter how you approach them. There are 2 main approaches:

- Server-side: let the database worry about data integrity and enforcing uniqueness constraints. If your constraint is expensive or you have hundreds of clients upserting, this operation will be slow: checking the constraint does not parallelize very well and requires a full  scan of the index created on the uniqueness constraint in some cases. You can see a great critique fo this approach here: https://github.com/apache/spark/pull/16692#issuecomment-274977343 . On some databases like Postgres, upserts require a UNIQUE index on the key or constraint to be upserted. That's just another operational detail to worry about.

- Client-side: let the client/code worry about data integrity. In this case, the client has to worry about identifying uniqueness constraints and merging records together. This can be quite error-prone. In addition, this approach means that  your database cannot guarantee the integrtiy of your data, any inconsistencies have to be resolved at the code level. You can see a good critique of this chalenges of this approach here: 
https://github.com/apache/spark/pull/16685 .

There are drawbacks everywhere, but for our case we chose the server-side aproach. The main reasons were:
- Using postgres to guarantee data ingrity was a must
- our constraints would be simple fields, which would not be too expensive for the  DB to check against

Generating records
--------------------
The easiest way to imagine how an upsert operation would take place would be to have an array of dicts, with the following constraints:

- all dicts in the array have the same fields
- the constraining field is present on all dicts
- the constraining field's values are unique. This is to avoid 2 or more workers updating the same record in the database at the same time. This will lead to conflicts and the upsert operation will be canceled

