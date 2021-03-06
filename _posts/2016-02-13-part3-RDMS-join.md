---
layout: post
title: Making sense of graph databases part 3 - How do RDMS do join?
enabledisqus: true

---

 _Main article can be found [here]({% post_url 2016-02-13-graph-databases %})._


A DBMS can join in many different ways and two of the most common ones are nested loop join and hash join.  All joins are performed two tables at a time, even if more tables are described in the query. This means that in cases with more than two join-tables, intermediary steps are required because the result of first join must be used joined with the third. This again impacts the performance.

## Nested Loop Join
Nested loop join is the simplest of the joins where the join is performed by, as the name suggests, nested loops. For each row in the inner table, all rows of the outer table are read. If the join is extended for more tables, the time complexity increases drastically. Nested loop join is only good for small tables or where the final result of the query is small. Indexes are used if they exist on the join column, and this will benefit performance. The following example for a nested loop join between two tables employees and departments, is taken from the Oracle [documentation](https://docs.oracle.com/database/121/TGSQL/tgsql_join.htm#TGSQL95232):

{% highlight sql %}
FOR erow IN (select * from employees where X=Y) LOOP
  FOR drow IN (select * from departments where erow is matched) LOOP
    output values from erow and drow
  END LOOP
END LOOP
{% endhighlight %}

If an index is useful in the outer loop due to the where condition or in the inner loop due to index on join column, it will be used. Regardless, the inner loop is run for each row from the outer loop. So, as the joins increase, the nested loops increase and when indexes are used, we avoid the worst case which is time complexity proportional to table sizes (`|employees|*|departments|`). But, even if indexes are used the complexity still increases and can increase non-linearally, which means it get slower and slower as more joins are added.

## Hash Join
Hash join prepares a temporary hash table of the smaller table before joining. It then traverses the larger table one time, while doing a lookup in the temporary hash table to find matches. For this to work best, the smaller data set should fit in main memory. Hash joins do not benefit from indexes on join columns because of the temporary hash table. The Oracle example for hash join is

{% highlight sql %}
FOR small_table_row IN (SELECT * FROM small_table)
LOOP
  slot_number := HASH(small_table_row.join_key);
  INSERT_HASH_TABLE(slot_number,small_table_row);
END LOOP

FOR large_table_row IN (SELECT * FROM large_table)
LOOP
   slot_number := HASH(large_table_row.join_key);
   small_table_row = LOOKUP_HASH_TABLE(slot_number,large_table_row.join_key);
   IF small_table_row FOUND
   THEN
      output small_table_row + large_table_row;
   END IF;
END LOOP;
{% endhighlight %}

Time complexity for hash joining two tables employees (larger) and departments (smaller), is the sum of:

1. Time for creating temporary hash table for departments  = must read all the rows in departments + create the hash table
2. For each row in employees, call the hash function to fetch the matching join row.

If more joins are added then this procedure must be repeated because of intermediary joins – i.e. create new hash table based on the result of current join to join with the third table. Based on this, it is possible to conclude that hash join gives linear complexity increase as the joins increase. However the actual time may be non-linear because of intermediary join steps, hash table constructions and hash lookups which are very dependent on the data. It is therefore not easy to give a general conclusion on how hash joins effect performance, but we do get an indication that it may or may not perform bad.

### Resources:
* <https://docs.oracle.com/database/121/TGSQL/tgsql_join.htm#TGSQL95232>
* <http://use-the-index-luke.com/sql/join>
