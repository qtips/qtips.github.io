---
layout: post
title: Making sense of graph databases part 2 - How do RDMS store and retrieve data with indexes

---

RDMS stores data record on disc I/O heap in blocks. To efficiently find the records, indexes are used which help locate the data, or else the system has to search every record until it finds what you are looking for. An index is any data structure that takes the value of one or more columns and finds the rows (record) with that value "quickly". Index can be compared to index back in a book; the key-words are sorted alphabetically and each keyword has corresponding page number(s) to where the word can be located.

Indexes are of two types: clustered and non-clustered. In clustered index, the data itself is stored on disc using a index storage scheme like B+ tree or hash index. Since the data is physically stored this way, there can only be one clustered index per table. Using a B+ tree in a clustered index, the data is stored on disc sorted on the index search key value (i.e. the column(s) you want an index on) as leaves of a B+ tree. The remaining nodes are used to perform search on the sorted data using the index search key. The cost of searching a B+ tree is `O(log(n))` and B+ trees are great for range queries since the data at leaf nodes is sorted -  so when a value is located, finding less than or greater than is done by simply gathering the leaves in either direction

 ![B+ Tree][btree]

Using clustered index with hash scheme, the data is stored as hash buckets without regards to any sort order. A hash function is used to map from index search key value to the correct bucket on disc. If the bucket contains more than one record, the bucket is scanned sequentially to find the correct record. Hash based index has a cost of `O(1)` for search, but are not effective for range queries.

Non-clustered indexes (a.k.a. secondary index) also use index schemes like B+ tree and hash index, but they do not effect how data is stored. For B+ tree and hash index, the leaves and buckets contain pointers to disc heap where the data is located. One can have multiple non-clustered index with different index search key for the same table [^1].

 [btree]: {{site.url}}/assets/bplustree.png "B+ tree. Source Wikipedia"

 [^1]: Ramakrishnan, Raghu, and Johannes Gehrke. Database management systems. Boston: McGraw-Hill, 2003. [slides from chapter 12](http://www.slideshare.net/koolkampus/ch12)