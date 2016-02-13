---
layout: post
title: Making sense of graph databases part 1 - When to use graph databases, and when not to
enabledisqus: true

---

 _Main article can be found [here]({% post_url 2016-02-13-graph-databases %})._

Intuitively, you would answer... graphs, but a bit more detailed information by people that have done some research would be nice. The Graph Database Wikipedia [article](https://en.wikipedia.org/wiki/Graph_database) says surprisingly little about this issue (feb 2016)! It only lists a huge amount of databases. This indicates that graph databases probably still are a little bleeding edge. But the Google results gave a lot of results for the Graph database Neo4j. They have in fact given out the O’Reilly book Graph Database which can be freely acquired from [Neo4j website](neo4j.com/books/graph-databases/). Although the book has huge focus on how Neo4j works, it still gives a good overview of what graphs in general and when the model is appropriate.

Graph databases are suited for highly connected data. These are the reasons I found for why graph databases outperform relational databases for these kinds of data:

1. **Better performance** – highly connected data can cause a lot of joins, which generally are expensive. After over 7 self/recursive joins, the RDMS starts to get really slow compared to Neo4J/Graph databases.
2. **Flexibility** – graph data is not forced into a structure like a relational table, and attributes can be added and removed easily. This is especially useful for semi-structured data where a representation in relational database would result in lots of NULL column values.
3. **Easier data modelling** – since you are not bound by a predefined structure, you can also model your data easily.
4. **SQL query pain** – query syntax becomes complex and large as the joins increase.

Points 2 to 4 are easy to agree with, at least with in my experience. The thing that caught my eye was the better performance feature of graph databases. I wanted understand what kind issues the joins are causing. This is not explained in detail in the book. To understand this I will look into how joins work in relational databases and why multiple joins cause performance issues in another [post]({% post_url 2016-02-13-part3-RDMS-join %}) .

## When RDMS is enough
It seems natural to also include use cases where graph databases are not well suited compared to RDMS.

When doing bulk and mass queries over a single table or tables requiring few joins, RDMS is a better choice. Consider the following examples
{% highlight sql %}
SELECT * FROM Person WHERE name='Tom'
SELECT * FROM Person WHERE age>10 AND age<20
{% endhighlight %}
In general RDMS will perform better at these queries because there is no relation. The first example is particularly true if both the RDMS and graph database do not use index on `name`, because RDMS will only look at all data in `Person` table while graph database must consider all nodes. For the range query, same rationale applies. In addition, using a B-tree index the range data is resolved quickly in the leaf level. Even with index on age labels, the graph database must find each node between age 10 and 20 separately.

Other reasons for using RDMS over graph databases:

* When the data is highly structured in predefined columns, i.e. the column values are not mostly null.
* RDMS have been around for 2 decades and are therefore
  * proven to work different kinds of environments and domains
  * well analyzed with standardized backing theory
  * stable and mature, in the sense that majority of bugs are sorted out
  * going to be around for some time because they are so popular (like COBOL is still used).

Note that I did not write the popular ACID guarantee as a feature for RDMS, because Neo4j and other graph databases do provide this feature as well.



