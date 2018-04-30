---
layout: page
title: All posts
excerpt: "All posts"
image:
    feature: #
---

<ul class="post-list">
{% for post in site.posts %}
  <li><article><a href="{{ site.url }}{{ post.url }}"><b>{{ post.title }}</b> by {{ site.data.authors[post.author].name}} <br/><span class="entry-date"><time datetime="{{ post.date | date_to_xmlschema }}">{{ post.date | date: "%B %d, %Y" }}</time></span></a></article></li>
{% endfor %}
</ul>
