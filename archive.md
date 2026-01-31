---
layout: page
title: Blog Archive
---

<link rel="stylesheet" href="/css/archive.css">

<div class="archive-tags">
{% for tag in site.tags %}
  <h2>{{ tag[0] }}</h2>
  <ul>
    {% for post in tag[1] %}
      <li>
        <span class="post-date">{{ post.date | date: "%B %Y" }}</span>
        <a href="{{ post.url }}">{{ post.title }}</a>
      </li>
    {% endfor %}
  </ul>
{% endfor %}
</div>