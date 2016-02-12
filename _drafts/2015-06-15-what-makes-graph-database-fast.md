---
layout: post
title: What makes graph database fast in dealing with linked data
---

Graph databases are gaining more and more popularity. Graph databases perform better on highly linked data which is a point illustrated by sketches of graphs compared with sketches of tables. It makes sense on the whiteboard, but how does it actually work in the computers?

 To answer this, we have to first look into how traditional databases work. Then we will compare the traditional approach with the graph database approach which will help identify what _actually_ makes graph databases so good.

# How Relational Databases work.

Lookups in relational database are best summarized with an example. Consider a database table `Person` with 10 000 rows and with the columns `firstname, lastname, address, age` and `sex`. The database does the following (roughly) when executing query

{% highlight sql %}
SELECT *
FROM Person
WHERE age = 25 AND lastname = 'Khan'
{% endhighlight %}

1. As the `Person` table is small, the database will read it from disc to memory
2. For each row in memory, it will check if the row contains lastname Khan
3. For each of rows with lastname Khan, it will check if the row contains age 25

In worst case, this takes *O(n<sup>2</sup>)* time because of two travels through all the rows. So far we have not considered indexes.

## Database Indexes

Indexes are used to pinpoint data fast instead of considering all data in table. An index is any data structure that takes the value of one or more columns and finds the rows with that value "quickly" (ref). Performing a query against and indexed column requires a search in the `index file` rather than on the table rows. This takes much less time and depends on the index structure used. Searching on a sorted index list takes *O(log(n))* (binary search) while using a hash mechanism takes O(1).

Quick note: Most popular index structure used is B-tree. It has a complexity of *O(log(n))* for search, however, it also supports range queries. On the other hand, hashtable based index has *O(1)* search complexity, but cannot be used for range queries.

Going back to our example query, using indexes will give much better time complexity. Consider that 427 people are aged 25 and 59 people have lastname Khan. If there is an index on age, the database will quickly fetch the 427 rows and than search for each of these to find Khan - time: index search + 427 rows. Similarly, if the index is on lastname, it will quickly find the 59 rows with Khan and check all of these for age = 25 - time: index search + 59 rows. However, if the index is on a combination of age and lastname, the relevant rows will be found simply through index search. In worst case, if an index is only on one of the columns, the complexity will be *O(n log(n))* using B-tree and *O(n)* using hash tables.

## Joins

Joining tables. In traditional databases, joining table
* Joining tables without index
* Joining tables with index.

In traditional  databases joining is done by using search. First First, a search is performed to find .



Assume we have new table `car` which lists cars together with the id of the personIf we want to joint hese ttwo tables, the table person and car, we have to find the person for each car. A better approach is to find car s for each person, as a person can have many cars while a car can only have one owner. ..

So assuming that a there are

Person table has 10 000 rows ,so we can assume that car table has 25 000 rows. Doing a join will then require the time for r. For each row in Person, The ar table will be searched. This will produce 10 000searches in car. . This measn that we have to search tthe car table of 25 000, 20 000 times. Generally speaking, tathis kind of joint akes O(nxm )time. . .  . This is a naive approach, a better way is to use indexes. By having an index on the car table’s person idcolumn (together iwth the primary key). This approach will be much faster. In this cawe, for each person row, the search in car table will me depenfaster, dependent on the index mechanism: B-tree: O(nxlog(m), hash: O(nx1)  .

It is useful to consider a three table sceario as well. If we assume that cars can be used by multiple persons, than we need a thrid table to describe  the car person relationship. This way, a car can have multiple users , while  a single owner can use multiple cars.. Again, we start with the index free approach. . The car-person table consist of 30 000 rows, while the car table consists of 50000 rows. Joining these three tables then becomes. Then consists of 10 000 x 30 000  opertaions for firs find all the ars the person uses. An for each cars used by the person, a search for the car must be conducted. Summing it up w eget:  (in worst cae) 10 000 x 30 000 x 5000. Generally, this becomes O(nxmxq). Adding indexes O(n x log(m) x log(q) ) and O(n).

Thus the general trend is that as we add more tables to join the time without index gets O(a1 xa2 x a3 … x ai). With index we get O( a1 x log(a2) x log(a2)…x log(ai) and O(a1x1…x1)

Joining in DBMS according to book.

## Several joins

* How does complexity evolve when including several joins

# How Graph databases work

* What happens when performing similar query with graph database?









* foaf example from Neo4J http://www.youtube.com/watch?v=UodTzseLh04 RDMS 2000 ms vs 2ms
* Relational database performance drops off markedly with more than seven JOINs.
* Relational database save typically save stuff in record on disk/memory. Time consuming stuff is to do read/write to I/O disk. Essentially, one wishes to reduce multiple calls to disk by knowing what to get and where to get it from in few operations as possible. Using database indexes traditional RDMS are able to fetch records in fewer operations because indexes allow them to pinpoint where they are located.
* When joining two table in RDMS, you have to search for maching values. The index may remedy going in one direction, however, going in opposite direction requires another index. (Example of join from book p. 387)
* Graph databases come in different flavors. Neo4J use index-free adjency to model relationships which allows for traversal of graph networks. Other graph databases rely on indexes, which may suffer from same type of limits as RDMS (7 join limit? Can be remedied with indexes?) (Graph Databases book)

Neo4J:
* storeage and lookup - hashing?: if we have a node with id 100, then we know its record begins 900 bytes into the file. Based on this format, the database can directly compute a record’s location, at cost O(1), rather than performing a search, which would be cost O(log(n)).
