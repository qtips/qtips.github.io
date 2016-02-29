---
layout: post
title: Find all UNIX commands/alias and their last modified timestamp
enabledisqus: true
---

Trying to clean up your unix? The following lists all commands and aliases in your terminal and sorts them based on their last modified date:
 
 {% highlight bash %}
 compgen -c | xargs type -a | awk '{print $3}' | xargs stat -f "%m%t%Sm %N"  | sort -rn | cut -f2-
 {% endhighlight %}

|Command|Explaination|
|-------|------------|
|`compgen -c`| list all commands/alias that can be used. Uses same mechanism as autocomplete|
|`type -a` | finds the commands´ location|
|`awk '{print $3}'`| prints the third word, which is the location of the commands´|
|`stat -f "%m%t%Sm %N"`| prints the last modified date of the file|
|`sort -rn`| sorts the input based on the last modified date (found from man page of stat!)|
|`cut -f2-`| trims parts of the output|