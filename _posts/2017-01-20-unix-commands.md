---
layout: post
title: Shell (unix) commands you always forget
enabledisqus: true
---

This is a list of typical commands that I often have to Google because I forget either their usage or important flags.

###  Using `grep` with perl type regex

{% highlight shell %}
ls | grep -P "dirNo\d\d$"
{% endhighlight %}

Using the `-P` flag lets you search for more advanced regex expressions. Other useful grep flags:

* `-i` ignore case
* `-o` print only matching
* `-v` invert matching
* `-A number` show lines after match
* `-B number` show lines before match

### Using `grep` without pipe

{% highlight shell %}
grep "hell\w" . -R
{% endhighlight %}

searches after pattern `hell\w` in current directory and sub-directories. Add `-h` to ignore filenames in output.

### For loops - range

{% highlight shell %}
for ((i = 1; i<=10; i++)); do echo $i; done
{% endhighlight %}

### For loops - command

{% highlight shell %}
for v in `ls`; do echo $v; done
{% endhighlight %}

### Bash script for traversing arguments and line breaks

{% highlight viml %}
#!/bin/bash

IFS=$'\n'
SENTENCES="line 1${IFS}line2${IFS}line3"

for arg in "$@" # for all input arguments
 do
 #automatically loops through line breaks duo to IFS value
   for s in $SENTENCES
     do
       echo "$arg=$s"
     done
 done
{% endhighlight %}


### Simple `sed` commands
Search and replace file content:
{% highlight shell %}
sed -i "s/<searchpattern>/<replace>/g" <files>
{% endhighlight %}

From pipe and print result:

{% highlight shell %}
sed -n "s/<searchpattern>/<replace>/p"
{% endhighlight %}

To replace matched regex groups, use \1, \2 and so on.

### Finding size of subfolders, sorted.

{% highlight shell %}
du -h -d 1 | sort -hr
{% endhighlight %}

* `du -h d -1` is the disc usage command with the `-h` flag for outputting human readable sizes and `-d 1` for depth of 1 sub dirs.
* `sort -hr` sorts human readable sizes in reverse order.

