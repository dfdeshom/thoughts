General Strategies for using Function Queries
======================================================

Function queries (http://wiki.apache.org/solr/FunctionQuery) are a convenient way to add relevance signals other than TF-IDF to a set of documents. You can use them as boost parameters or sort parameters to you queries. I assume you're already familliar with implementing them in in Java, my favorite example is here: http://www.supermind.org/blog/1042/using-mongodb-from-within-solr-for-boosting-documents. I'm simply going to poin out some tips when considering them. 

Efficient use of function queries
---------------------------

There are 2 parameters to consider if you want a function query to return results in a reasonable time:

* the number of documents to search over in Solr. The larger the number, the slower the query will be. The only sure way to reduce this number is to use filter queries. For example, for a Solr index wither over 500K documents with a ``pub_date field``, we could consider applying a function query over documents published in the last 2 weeks only, which would reduce the number of documents to search over to say, around 100-500. This approach makes sense fo function queries that use pageview data as a signal.

* The number of signals available for each document. Not all documents will have a signal. For example, for our index with 500K documents, only about 1K or so may have page view data associated to them. Considering these 1K documents makes more sense. This is the approach we have been using so far at work.

Because of the limitations above,a function query for boosting by popularity will only make sense if it’s defined over a tight interval like a ``pub_date`` interval, or a combination of ``section`` and ``author``, or a combination of all three.

Speed over correctness
------------------------

Our approach to writing function emphasizes speed over “correctness”. Since we are not returning any of these signals explicitly, we can afford to have them be slightly off for some period of time.

Here’s the general pattern of writing a function query so far:

* Write a function that retrieves a “reasonable”/representative number of signals and cache these urls. 

* Refresh the cache in the background by periodically querying for top URLs. This ensures that new data shows up eventually while we’re serving recommendations.

The above approach means that the first query will take time, but subsequent ones will always respond fast, until the Solr instance is restarted.

We chose against using a distributed cache like Redis because of our need for speed. Function queries in Solr are implemented such that we need a cache with fast bulk insertions and fast individual lookups. There is no option of bulk lookups (this is a limitation of the way Lucene works). For now, the advantages of using an individual cache for each Solr instance outweighs the drawbacks.

Have an API to access your data
--------------------------------

Chances are your signals are in different databases. It will help greatly if have an API that does the heavy work for you. The reasoning behind that is that you want the code that's fetching these signals to do as little as possible in as uniform as a way as possible. 

Ideally, the function query should do simple HTTP or database request from the API and parse and store the result. No one wants to re-write complicated mongodb queries in Java. If the data needed is not in the API, you should just add it.

Prefer methods of population that are bulk-oriented. It’s much faster to do a query asking for 500 items that 500 queries asking for one item.

