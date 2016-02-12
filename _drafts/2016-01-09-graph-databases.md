---
layout: post
title: Graph Databases
---

Having worked with Virtuoso graph database for half a year I was excited to be given opportunity to try something new. Although my work with the database involved mainly querying with SPARQL and some graph modeling, it caught enough interest that I wanted to understand a bit more. My main question was how, exactly, does Graph databases perform better than relational databases for graphed data. Is it how it stores the data that makes it so good? Or are there just some algorithm-magic that makes graph databases super slick. While pondering over these question, I understood how little I knew about graph databases. In fact I don’t even know when to use graph databases (over relational databases). This resulted in several blog posts:

* First I wanted to find out the [use cases of graph databases]({% post_url 2016-01-04-when-to-use-graph-databases %})
* Then, to be able to compare graph database to relational database performance, I needed to understand how relational database [stores and retrieves]({% post_url 2016-01-05-RDMS-index %}) its data, and [how joins work]({% post_url 2016-01-06-RDMS-join %})
* Finally, I [compare]({% post_url 2016-01-07-rdms-vs-graph %}) relational database approaches with the native graph database Neo4j to understand why Neo4j works more efficiently on connected data than relational databases
* While doing my research, I understood that that the graph database field is still in development, so much that the term _graph database_ is not even standardized, and that that there are actually [three kinds of databases]({% post_url 2016-01-08-types-of-graph-databases %}). Virtuoso, for example, is more popularly called a _triple store_.


