A shorter
http://www.datastax.com/dev/blog/basic-rules-of-cassandra-data-modeling

Wide rows only
----------------
There's only one way scale in C*: use wide rows

But not too wide
------------------
Small catch: C* starts choking when you have more than 100k columns on a single
row and want to read it back

Know your answers beforehand
------------------------------
You should use C* when you know the answer to your questions
beforehand but are too lazy to look them up. People also call that "knowing your queries beforehand"

Traps
-------
As a cruel joke, C* developpers have decided to add other structures
like counters and sets even though they don't scale well at all. Never
make the mistake of using them. If you find that you need them, you
may need to either:
- re-think your problem. Use more wide rows, but not too wide.
- rejoice in the fact that you actually don't have a "big data" problem at
  all. Use Postgres, it will work infinitely better.

The Unix way
--------------
In conclusion, you could argue that C* makes scaling easy since
there's only one way to model your data: fit as many of it into as few
rows as possible. That somewhat fits the Unix of doing one thing
well. 
