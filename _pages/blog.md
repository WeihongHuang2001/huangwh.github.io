---
layout: default
title: "Tech Blog"
permalink: /blog/
---

# 📝 Tech Blog

Here I share my technical notes, learning summaries, and research insights.

{% for post in site.posts %}
<div class="blog-card">
  <h3><a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></h3>
  <p class="blog-meta">
    <time>{{ post.date | date: "%Y-%m-%d" }}</time>
    {% if post.tags %}
      {% for tag in post.tags %}
        <span class="tag">{{ tag }}</span>
      {% endfor %}
    {% endif %}
  </p>
  <p class="blog-excerpt">{{ post.excerpt | strip_html | truncate: 200 }}</p>
  <a href="{{ site.baseurl }}{{ post.url }}">Read more →</a>
</div>
{% endfor %}
