---
layout: default
title: Tom Schaefer
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | prepend: site.github.url }}">
        {{ post.title }}
      </a>
    </li>
  {% endfor %}
</ul>
