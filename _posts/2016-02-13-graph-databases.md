---
layout: post
title: Making sense of graph databases
enabledisqus: true
---

Having worked with Virtuoso graph database for half a year I was excited to be given opportunity to try something new. Although my work with the database involved mainly querying with SPARQL and some graph modeling, it caught enough interest that I wanted to understand a bit more.

My main question was how, exactly, do graph databases perform better than relational databases for graphed data. Is it how they store the data that makes them so good? Or is there just some algorithm-magic that makes graph databases super slick. While pondering over these question, I understood how little I knew about graph databases. In fact I don’t even know when to use and not use graph databases.

While reading up, it took a while before I understood that there are _at least_ two kinds of graph databases: Graph databases based on relational databases (like _triple stores_) and _native_ graph databases. My analysis is based on comparing native graph databases (primarily Neo4j) with relational databases, but in the [last part]({% post_url 2016-02-13-part5-types-of-graph-databases %}), I present triple stores.

My questions resulted in several blog posts:

* First I wanted to find out the [use cases of graph databases]({% post_url 2016-02-13-part1-when-to-use-graph-databases %})
* Then, to be able to compare graph database to relational database performance, I needed to understand how relational database [stores and retrieves]({% post_url 2016-02-13-part2-RDMS-index %}) its data, and [how joins work]({% post_url 2016-02-13-part3-RDMS-join %})
* Finally, I [compare]({% post_url 2016-02-13-part4-rdms-vs-graph %}) relational database approaches with the native graph database Neo4j to understand why graph databases works more efficiently on connected data than relational databases. I also look at related work regarding Neo4j performance.
* While doing my research, I understood that that the graph database field is still in development, so much that the term _graph database_ is not even standardized, and that that there are at least [two kinds of graph databases]({% post_url 2016-02-13-part5-types-of-graph-databases %}). Virtuoso, for example, is more popularly called a _triple store_.


