---
layout: post
title: Running shell commands with Python
enabledisqus: true
---

Although bash scripts are powerful, the language contains many gotchas that makes your head scratch. I therefore really enjoy the simplicity of Python. The following is a simple way to run shell command and get the result, using Python. Note that output to stderr will not be returned.

{% highlight python %}
import subprocess

    def run_cmd(command):
        return subprocess.Popen(command, shell=True, stdout=subprocess.PIPE).stdout.read()

{% endhighlight %}

Usage example - find maven dependencies, do some filtering and print the result:

{% highlight python %}

dependencies='mvn dependency:list -DexcludeTransitive=true | grep -v Download | grep -v test | sed -n "s/^[^:]*:\([^:]*:.*\)/\\1/p" | sort'
print run_cmd(dependencies).split()

{% endhighlight %}


