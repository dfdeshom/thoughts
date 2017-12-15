Upserting rows in postgresql 9.5 through spark
===============================================

Pie in the sky
----------------
Although the current Postgres JDBC data source allows SELECT and INSERT operations with Spark, it doesn't allow for upserts. Part of the reason is that SQL database support for upsert operations varies a lot, up until recently Postgres did not even support it. The other reason is that upserts are not generally efficient at scale no matter how you approach them. There are 2 main approaches:

- Server-side: let the database worry about data integrity and enforcing uniqueness constraints. If your constraint is expensive, this operation will be slow: checking it does not parallelize very well and requires a full  scan of the index created on the uniqueness constraint. You can see a great critique fo this approach here: https://github.com/apache/spark/pull/16692#issuecomment-274977343 . On some databases like Postgres, upserts require a UNIQUE index on the key or constraint to be upserted. Thst's just another operational detail to worry about.

- Client-side: let the client/code worry about data integrity. In this case, the client has to worry about identifying uniqueness constraints and merging records together. This can be quite error-prone and in addition your database cannot guarantee the integrtiy of your data. You can see a good critique of this chalenges of this approach here: 
https://github.com/apache/spark/pull/16685 .


