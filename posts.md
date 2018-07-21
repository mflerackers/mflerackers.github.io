---
title: Blog
layout: single
author_profile: true
read_time: false
comments: false
share: true
related: false
permalink: posts
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
