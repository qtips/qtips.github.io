---
layout: post
title: What are the use cases of Graph Databases?
---

Intuitively, you would answer... graphs, but a bit more detailed information by people that have done some research would be nice. The Graph Database Wikipedia [article](https://en.wikipedia.org/wiki/Graph_database) says surprisingly little about this issue (feb 2016)! It only lists a huge amount of databases. This indicates that graph databases probably still are a little bleeding edge. But the Google results gave a lot of results for the Graph database Neo4j. They have in fact given out the O’Reilly book Graph Database (link) which can be freely acquired from [Neo4j website](neo4j.com/books/graph-databases/). Although the book has huge focus on how Neo4j works, it still gives a good overview of what graphs in general and when the model is appropriate.

Graph databases are suited for highly connected data. These are the reasons I found for why graph databases outperform relational databases for these kinds of data:

1. Better performance – highly connected data can cause a lot of joins, which generally are expensive. After over 7 self/recursive joins, the RDMS starts to get really slow compared to Neo4J/Graph databases.
2. Flexibility – graph data is not forced into a structure like a relational table, and attributes can be added and removed easily. This is especially useful for semi-structured data where a representation in relational database would result in lots of NULL column values.
3. Easier data modelling – since you are not bound by a predefined structure, you can also model your data easily.
4. SQL query pain – query syntax becomes complex and large as the joins increase.

Points 2 to 4 are easy to agree with, at least with in my experience. The thing that caught my eye was the better performance feature of graph databases. I wanted understand what kind issues the joins are causing. This is not explained in detail in the book. To understand this I will look into how joins work in relational databases and why multiple joins cause performance issues. (TODO link1)




