---
title: ブログ
id: my_blog
layout: default.ja
---

ブログ
=====

こちらで小さな記事を発表しています。役に立つかもしれません。どうぞ自由にこの情報を使ってください。

目次
---

{% assign posts=site.posts | where:"lang", page.lang %}
{% for post in posts %}
 * [{{ post.title }}]({{post.url}})
{% endfor %}
