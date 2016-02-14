---
layout: post
title: Virtuoso SPARQL Replace Function Bug and Workaround
enabledisqus: true
redirect_from: /virtuoso-SPARQL-replace-bug
---

Running the following SPARQL for replacing `%c3%85` with the letter `Å` runs as expected

{% highlight sparql %}
select REPLACE("%c3%85-XYZ-%20%28-DEF-%29","%C3%85", "Å", 'i')
where {}

#Result: Å-XYZ-%20%28-DEF-%29
{% endhighlight %}

However, when using nested `REPLACE` statements with an outer replace having a regex with `.`, the replace function "jumps" back one character where the match is found :

{% highlight sql %}
select REPLACE(
             REPLACE("%c3%85-XYZ-%20%28-DEF-%29","%C3%85", "Å", 'i'),
          "%..(%..)*", "?", 'i')

where {}
# Result: Å-XYZ?8-DEF?9
# Expected: Å-XYZ-?-DEF-?
{% endhighlight %}

This only happens for some replace characters including all of `ÆØÅæøå`. Workaround for this is to run a `CONCAT` before the second `REPLACE`, which seems to "reset" the string before sending it to next `REPLACE`:
{% highlight sparql %}
select REPLACE(

CONCAT(REPLACE("%c3%85-XYZ-%20%28-DEF-%29","%C3%85", "Å", 'i'),""),

"%..(%..)*", "?", 'i')

where {}

# Result: Å-XYZ-?-DEF-?
{% endhighlight %}

This was tested using Virtuoso version 07.20.3212 on Linux (x86_64-unknown-linux-gnu), Single Server Edition with Virtuoso SPARQL Query Editor

I also made an [issue](https://github.com/openlink/virtuoso-opensource/issues/415) on Github at Openlink/Virtuoso in case they fix this :)
