---
layout: post
title: Back to basics - how does RDMS store and retrieve data with indexes

---

RDMS stores data record on disc I/O heap in blocks. To efficiently find the records, indexes are used which help locate the data, or else the system has to search every record until it finds what you are looking for. An index is any data structure that takes the value of one or more columns and finds the rows (record) with that value "quickly". Two commonly used index schemes are hash-based and B-tree based. Hash based index uses the column-value in a hash function to get the rows’ heap location on disc I/O. This gives a time of `O(1)` using Big-O notation. Hash based index performs really well for queries where the conditions are equal to some static value, but are not so good for range conditions. For these, its better to use B-tree. B-tree based index creates a tree out of the column values so doing a binary search is sufficient to find the data using `O(log(n))` time. In addition, the leaf nodes of the tree are sorted which is great for supporting range queries – so when a value is located, finding less than or greater than is done by simply gathering the leaves in either direction. B+tree is probably best explained by an image:

![B+ Tree][btree]

[btree]: {{site.url}}/assets/bplustree.png "B+ tree. Source Wikipedia"