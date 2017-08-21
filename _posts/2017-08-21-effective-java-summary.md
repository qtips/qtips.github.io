---
layout: post
title: Effective Java Summary
enabledisqus: true
---

Here is a collection of short summaries of Joshua Blochs great book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) for easy future reference.

{% for post in site.posts reversed %}
  {% if post.tags contains 'effectivejava' %}
[{{ post.title }}]({{post.url}})
  {% endif %}
{% endfor %}