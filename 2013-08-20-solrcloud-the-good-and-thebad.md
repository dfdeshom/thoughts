We're coming up to a year now since SolrCloud (Solr 4.0) has been released. The company I work for has recently switched to Solr 4.3 and the overall impression has been good, although there has been some growing pains. What follows are my impressions about what I've liked and not liked so far about SolrCloud

The Bad
------------

You can still run Solr in "non-cloud" mode. This means that there are 2 code paths in the ``lucene-solr`` repo. It also means that support questions can get a little more complicated. There are a some issues that come up because of this separation:

* Configuration is somewhat in flux. The ``solr.xml`` file is scheduled for a major change in Solr5 (http://wiki.apache.org/solr/Solr.xml%204.4%20and%20beyond) and might completely disappear. ``schema.xml`` and ``solrconfig.xml`` now live in Zookeeper.

* There seems to be some confusion over the cores API and the collections API. The collections API is a nice superset of the cores API but some think they can be used interchangeably. People using Solr in cloud mode should use the collections API, people using Solr in non-cloud mode should be using the cores API.

* Support for large number of homogeneous cores doesn't mean support of large number of homogeneous collections (http://wiki.apache.org/solr/LotsOfCores). There are some nice options and performance improvements that make managing a large number of cores possible in Solr, unfortunately, they are only available in non-cloud mode. 


The Good
--------------

High availability via Zookeeper has been good.

The performance improvements have also been impressive. They have lead to smaller index sizes, reduced memory usage and faster indexing speed. These things alone are what makes upgrading to the 4.x series attractive.

The introduction of function queries API mean that it's relatively easy to write boost functions that take into account external signals that change frequently. There were 2 common solutions to indexing these signals before that:

* re-index the signal every ``n`` minutes.  

* shard the index by date and index a new signal into documents for each shard.

These solutions are still good but I like the simplicity the function queries API gives you.

Overall, upgrading to SolrCloud has been a great thing for us. I'm looking forward to Solr 5 which should have a cloud mode only and more performance gains from both Lucene and Solr.
