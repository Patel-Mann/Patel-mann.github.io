---
layout: default
title: Blog
---

<h1>Blog</h1>

<p>Thoughts, tutorials, and experiences.</p>

<ul>
  {% for post in site.posts %}
  <li>
    <a href="{{ post.url }}">{{ post.title }}</a>
    <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span>
  </li>
  {% endfor %}
</ul>