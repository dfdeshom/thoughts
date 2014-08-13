Prioritiy queue issues w/ Kafka
=================================

We are trying to implement a priority queue using only Kafka. This is part of a larger project to make Scrapy use Kafka as scheduling backend. One of the nice-to-haves (if not necessities) is to have a priority queue so that you can put some requests in front of other if necessary.

So far, I've identified 2 ways to do this: multiple partitions or multiple topics

Multiple partitions
--------------------

One caveat: 
   > Kafka only provides a total order over messages within a partition, not between different partitions in a topic
   
from http://kafka.apache.org/documentation.html


This could be the ideal way, as Kafka writers say to prefer fewer larger topics with multiple partions over small topics:https://cwiki.apache.org/confluence/display/KAFKA/FAQ#FAQ-HowmanytopicscanIhave? Further, they say:

    > The actual scalability is for the most part determined by the number of total partitions across all topics not the number of topics itself 
    
which is good news if we're considering multiple topics instead. The drawback of this approach is that our number of partions needs to be known beforehand, so we can't create partions on-the-fly. Imagine you have a list of requests with priorities `[1,7,123]`. How do you map these numbers to partitions without using some bucketing? There is no upper bound to a priority number, and we can't have infinite partition numbers.

Here are the possible schemes with multiple partions

- bucketing: this is the easiest one: classify each request into 3 buckets: high, low, normal and use some cut-off point to classify those. Use a partitioner to put messages in right partition. This isn't really a priority queue, but might be good enough

- replicate the request over a fraction of total partition based on priority. The idea is that if you have 5 partitions, and have a request with high enough priority, you could write this message to the topic accrross 4 partitions, for example. this would increase the likelyhood that the request gets processed faster. We would have to deal with what hapens when the same request gets picked up by 2 different spiders. 


- what if we capped priorities at something reasonable, eg -100 to 100?  This is probably what we'll have to do.

Multiple topics
-----------------

The idea is to have as many topics as there can be priorities, with each topic having just one partition. One obvious drawback is that the Consumer would have to constantly query the Zookeeper machine for the topic names since they can change a lot. Again, as sane cap on the number of priorities would help.

Ops stuff
----------

- number of partitions will need to be configured

- log retention ideally should be really short, 1hour could be enough




