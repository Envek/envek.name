---
title: My blog
id: my_blog
layout: default.en
---

My blog
=======

Here I'm publishing various articles that are too small for special resources but can be useful by itself.


Index
-----

{% assign posts=site.posts | where:"lang", page.lang %}
{% for post in posts %}
 * [{{ post.title }}]({{post.url}})
{% endfor %}
