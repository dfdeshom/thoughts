What's wrong with Solr command-line tools?
===========================================

Usability of Solr has been improving at a glacial pace, although it has been getting better. I couldn't quite put my finger with what was missing from it, until Andrew made the remark that people are simply not treating Solr/ElasticSearch/Lucene as a database. In fact we were making fun of people in the ElasticSearch community advocating storing data directly on it, instead of syncing it to a database periodically.


But then, we thought: Lucene is actually a database, even if it cannot be the only database you rely on for your data. So, why do we have no database console tools for interacting with Solr? Why do I have to fire up a web browser every time? ElasticSearch is doing much better in this case: it has different frontends, although they are all web-based: http://www.elasticsearch.org/guide/en/elasticsearch/client/community/current/front-ends.html . But still, not much of a command-line tool.

We've been spoiled by mongodb's ecosystem: it has been extremely dev-friendly with great clients and a great console. I want the same thing for Solr, especially a great console where one can do everything the web interface does.

This will get easier and easier as more and more of  Solr can now be controlled via its web API. I'm thinking here of:

- the collections API, which has improved a lot
- the schema API, https://wiki.apache.org/solr/SchemaRESTAPI . Still needs some tweaks
- the cores API, which I hope will disappear soon

People writing clients for Solr should also assume it's a database. For example, Solr needs to be configured such that clients never need to call `commit` and `optimize`. In fact, this seems like a mistake now that Solr has `autocommit` and `autosoftcommit` settings. Clients should just forget about this API call and assume the server configuration for committing data is sane. 

It might not be bad idea to have a distribution of Solr with a striped-down, less flexible `solrconfig.xml`. There are many moving parts within this file. The same could be applied to `schema.xml` and making it optional will help.

Overall, we need to start seeing Solr as a "real" document database and expect to interact with it on the command line like we would with mongodb.
