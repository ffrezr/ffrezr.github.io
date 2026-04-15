---
layout: page
title: Blog
permalink: /blog/
---

## Posts

{% for post in site.posts %}
- **[{{ post.title }}]({{ post.url | relative_url }})** — {{ post.date | date: "%B %d, %Y" }}

  {{ post.excerpt | strip_html | truncatewords: 30 }}

{% endfor %}

{% if site.posts.size == 0 %}
Coming soon! In the meantime, check out my articles on [Medium](https://medium.com/@franciscofrez).
{% endif %}
