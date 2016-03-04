---
layout: post
title: Why are JDBC connections slow?
enabledisqus: true
---

The very first connection is always very slow (for MySQL it took over 300 ms) because of class initializations and creations of singletons. Therefore, the subsequent connections are used.

Profiling is used because it shows how each call performs. Profiling does have its overhead because events are sent when entring a method and when exiting. First I tried to profile with all classes, which showed exactly what methods were called and each call´s time. However, this approach has too high overhead giving unreliable time readings. Profiling with all classes enabled is therefore
primarily useful for getting an overview of what classes are involved, especially if you are profiling someone elses code. When you have this overview, profiling with important classes can be performed to get actual time readings. A useful way to get a glimpse of the profiling overhead is to compare print time before and after a high level call with the profiling times. This approach helps get reliable times for code execution using profiling.

[NetBeans]() with Java VisualVM is an excellent tool to perform profiling because it allows easier profiling right from the code. To avoid the initialization run destroying the measurements, [Profiling Points]() with resets were used.

For comparison, I though it can be useful to compare the database connection time with doing simple http requests. BUT, http connections are persistent - meaning that existing connections to a client are reused. From Oracle [documentation](http://docs.oracle.com/javase/6/docs/technotes/guides/net/http-keepalive.html):
> (...) unless otherwise indicated, the client SHOULD assume that the server will maintain a persistent connection, even after error responses from the server.

So, it will be difficult to separate cost of actual connection from class initialization/singleton creations. Any http requests after the first will avoid the initialization costs as intended, but they will also reuse the first connection. I tried to set the system property ´keep-alive´ to false to disable persistence, which did create slower connections. But because the server already has opened a [socket](http://docs.oracle.com/javase/tutorial/networking/sockets/definition.html) for me as a client, that socket is reused when I reconnect, making connections faster. The http request is significantly slower (50%) when I reconnect after a couple of minutes, which indicates that a timeout at server-side has deleted my clients socket.

So jdbc connection comparison with http request give some indication, but there are some gotchas! The http requests to some local  servers took 30-60 ms with keep-alive disabled. With keep-alive pooling enabled, it took approximately 20 ms. Time was measured using system clock.

From the same article, it is interesting to note why the persistence is useful:


>* Network friendly. Less network traffic due to fewer setting up and tearing down of TCP connections.
>* Reduced latency on subsequent request. Due to avoidance of initial TCP handshake
>* Long lasting connections allowing TCP sufficient time to determine the congestion state of the network, thus to react appropriately.

TODO for this post:
1. Kjøre JDBC kall med Connection Pooling fra glassfish server 1t
2. Sammenligne tall med og uten pooling, og med http request
3. Gjør samme test med en annen database eks sqlite eller postgresql
4. Skrive: kode ekspempler, sammenligne tid med profilering slått av, sammenligne tid med profilering på på hovedklasser, sammenligne med http pooling på og av, konklusjon
5. Fylle ut lenker


MySql:

With profiling disabled, it took appoximatly 4 ms to connect to the database, when measuring time withing code:
{% highlight java %}
#TODO
{% endhighlight %}

When performing a initial connection through the driver it takes approximatly 17 ms , the following snapshot is seen:
  <image>

The stuff that takes time are:
TODO REVISE!
* connection and driver property initialization (different kinds of string parsing and stuff) 6 ms
* the actual connection with handshake 3.5ms
* initilizing properties from server  5.5ms - which is doing the following query:
{% highlight sql %}
/* mysql-connector-java-5.1.38 ( Revision: fe541c166cec739c74cc727c5da96c1028b4834a ) */
SELECT
@@session.auto_increment_increment AS auto_increment_increment,
@@character_set_client             AS character_set_client,
@@character_set_connection         AS character_set_connection,
@@character_set_results            AS character_set_results,
@@character_set_server             AS character_set_server,
@@init_connect                     AS init_connect,
@@interactive_timeout              AS interactive_timeout,
@@license                          AS license,
@@lower_case_table_names           AS lower_case_table_names,
@@max_allowed_packet               AS max_allowed_packet,
@@net_buffer_length                AS net_buffer_length,
@@net_write_timeout                AS net_write_timeout,
@@query_cache_size                 AS query_cache_size,
@@query_cache_type                 AS query_cache_type,
@@sql_mode                         AS sql_mode,
@@system_time_zone                 AS system_time_zone,
@@time_zone                        AS time_zone,
@@tx_isolation                     AS tx_isolation,
@@wait_timeout                     AS wait_timeout
{% endhighlight %}
