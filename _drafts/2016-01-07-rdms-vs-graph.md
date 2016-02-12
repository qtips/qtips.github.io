---
layout: post
title: Why are Graph Databases more efficient than RDMS on connected data
---
In a [previous post]({% post_url 2016-01-04-when-to-use-graph-databases %}), it was argued that graph databases are superior to RDMS on connected data because, RDMS has to do lots of joins. In this post we will look into why. 
  
So, why are joins not a problem for Neo4j? The neat thing about Neo4j is that it does not use indexes at all. Instead, graph nodes are linked to its neighbours directly in the storage. This property of reference is called *index-free adjacency*. So, when Neo4j performs a graph query for retrieving a sub-graph, it simply has to follow the links in the storage – no index lookups are needed. The cost of each node-to-node traversal is `O(1)` – so if we are doing a friend-of-a-friend look-up to find, for example, all indirect friends of Alice, the total cost would be proportional to total number of friends in the final result  – `O(m)` where `m` is the total number of Alice’s indirect friends. Even if the data (number of nodes) increases, if the total number of friends are still the same, the cost will still by `O(m)`. This is because Neo4j first “jumps” to the Alice node using some kind of search or index, and then it simply visits the relating nodes by following the links, until the query is satisfied [^1].

To compare the graph database approach with RDMS we will go through a simple example. In a [previous post]({% post_url 2016-01-06-RDMS-join %}), join types used in the comparison below were examined, together with their algorithms, which the Big-O number used in this post are related to.

Assume that we have tables Employees (E), Departments (D) and Payments (P).

![RDMS Example][employee_rdms]

By joining E with D and then E with P, we can calculate for example sum payment for a given department. Given departments `x` and `y`, we want to find all the payments to its employees. By using nested loop join, this approach has a cost of `O(2 * |E| * |P|)` (2 for two departments, as we assume that an hash index is used to retrieve the x and y). Using hash join, this gives a cost of `O(2 +|E|+2*|E|+|P|)`, where `2+|E|` is the first join, and `2*|E|+|P|` is the second. Again, the first 2 is for retrieving `x` and `y`, while `|E|` is for looping the larger table. `2*|E|` is the number of maximum rows that can be constructed by joining `x` and `y` with `E`. The `|P|` is the final looping of each row to find corresponding employee in the first joined set.

A *subset* graph of this example is as follows:

![Graph Example][employee_graph]

where *e's* are employees and *p's* are payments.

 To find the payments with Neo4j, first a search is conducted to find the departments `x` and `y`. Using hash indexing this gives `O(1)`. Then the graph is “walked” to find all the relevant payments, by first visiting all employees in the departments, and through them, all relevant payments. If we assume that the number of payments results are `k`, then, as mentioned in the Graph Database book [^1], this approach takes `O(k)`. We now have the following costs:

* Nested Loop join: `O(2 * |E| * |P|)`
* Hash Join: `O(2 +|E|+2*|E|+|P|)`
* Neo4j: `O(k)`

We see that as the number of employees and payments increase, the nested loop join increases the most, while the hash join also increases. However, Neo4j cost is not dependent of the amount of data, thus it stays the same. This analysis shows how Neo4j, in theory, will perform better as the data increases.

## Related Work
Analysis in this post is of course a very simple one, and was mostly conducted to understand the underlying details. In my research I did a survey find others say about performance.
[TODO]
Some other works on performance measure
Using the fruit of my masters degree, searching Google and Google Scholar, I managed to find several interesting articles and posts regarding Neo4j type graph database performance:

* https://baach.de/Members/jhb/neo4j-performance-compared-to-mysql/view

TODO: Add links for papers

[employee_rdms]: {{site.url}}/assets/employee_rdms.png "RDMS Example"
[employee_graph]: {{site.url}}/assets/employee_graph.png "Graph Example"

[^1]: Robinson, I., Webber, J. & Eifrem, E. (2013). [Graph databases](http://graphdatabases.com). Sebastopol, Calif. Sebastopol, CA: O'Reilly Media,O'Reilly Media, Inc.
