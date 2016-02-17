---
layout: post
title: Making sense of graph databases part 5 - Types of graph databases
enabledisqus: true

---

_Main article can be found [here]({% post_url 2016-02-13-graph-databases %})._

When doing my research on how graph databases work, I eventually discovered that there are at least two kinds of graph databases that differ on how they store and process graph data. I was surprised because I thought that there could not be many optimal ways of creating a graph database, because I was comparing them to relational database theory which has been standardized for quite some time. This simply means that graph databases are still a little bleeding edge. In this post I will present the different types of graph databases and say a little about how they work.


The two types I found were:

1. Triple store based on RDF
2. Native graph based on property graph model

## Triple store
* These databases were developed with aim to support the SPARQL query language from the [Semantic Web](https://en.wikipedia.org/wiki/Semantic_Web) movement.
  * This allows of using features like ontologies and reasoning using OWL.
* Often, they store all data in a single relational table with columns _subject_, _predicate_, _object_ corresponding to [RDF](https://en.wikipedia.org/wiki/Resource_Description_Framework) triples. Sometimes, a _graph_ column is also added to relate triples to a named graph.
* Virtuoso falls under this category.
* Triple stores often generates huge tables. To traverse a the graph the database has to do self-joins, which can kill performance.
  * Since triple stores use joins to connect graph nodes, they have same [disadvantages]({% post_url 2016-02-13-part4-rdms-vs-graph %}) as relational databases on graph data. Therefore clever tuning and indexing is required, but the performance is still not comparable with native graph databases (I have not found a performance comparison, though).
  * One approach (which is used by Virtuoso) is to use B+ trees together with [bitmap based indexing](https://en.wikipedia.org/wiki/Bitmap_index) on different combinations of the 4 columns - this can generate upto 16 indexes for the single table. To avoid that the indexes take more space than the actual data, duplicate rows are often reused, together with compression of index files. Compression works is extremely well on bitmap index! [^1]

## Native graph
* Called native graphs because they store data on disc in a graph oriented format that benefits graph query performance, instead of "simulating" a graph on a relational database.
* They often have proprietary query language, instead of a standard.
* Examples: [Neo4j](https://en.wikipedia.org/wiki/Neo4j), [Dex](https://en.wikipedia.org/wiki/DEX_(Graph_database)), [HypergraphDB](http://hypergraphdb.org/)
* A recent (2015) survey of such databases can be found [here](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?tp=&arnumber=7148480&url=http%3A%2F%2Fieeexplore.ieee.org%2Fxpls%2Fabs_all.jsp%3Farnumber%3D7148480)



[^1]: Luo, Yongming, et al. "[Storing and indexing massive RDF datasets.](http://www.win.tue.nl/~yluo/seeqr/files/11survey.pdf)" Semantic search over the web. Springer Berlin Heidelberg, 2012. 31-60.
[^2]: Robinson, I., Webber, J. & Eifrem, E. (2013). [Graph databases](http://graphdatabases.com). Sebastopol, Calif. Sebastopol, CA: O'Reilly Media,O'Reilly Media, Inc. Can also use this blog [post](http://neo4j.com/blog/other-graph-database-technologies/)




