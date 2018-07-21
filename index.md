---
title: Home
layout: single
author_profile: true
read_time: false
comments: false
share: true
related: false
---

## Math for game development

<ul>
{% for chapter in site.math %}
  <li>
    <a href="{{ chapter.url }}">{{ chapter.title }}</a>
  </li>
{% endfor %}
</ul>

## Blog

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
