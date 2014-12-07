---
title: Мой блог
id: my_blog
layout: default.ru
---

Мой блог
========

Здесь я выкладываю разные статьи, которые слишком малы, чтобы попасть на профильные сайты, но определённо могут быть полезны.

Содержание
----------

{% assign posts=site.posts | where:"lang", page.lang %}
{% for post in posts %}
 * [{{ post.title }}]({{post.url}})
{% endfor %}
