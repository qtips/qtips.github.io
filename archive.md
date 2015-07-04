---
layout: page
title: Archive
---

## Blog Posts

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }}) <!--:<small>{{ post.content | strip_html | truncatewords: 10 }}</small>-->
{% endfor %}