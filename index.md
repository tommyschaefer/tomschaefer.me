---
layout: default
title: Tom Schaefer
---

<ul class="posts">
  {% for post in site.posts %}
    <li class="post">

      <span class="date">{{ post.date | date: '%B %d, %Y' }}</span>

      <a href="{{ post.url | prepend: site.github.url }}">
        <h1>{{ post.title }}</h1>
      </a>

      <p>
        {{ post.summary }}
      </p>
    </li>
  {% endfor %}
</ul>
