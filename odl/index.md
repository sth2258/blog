---
title: Welcome to The Engineering Architect's Perspective
---
Welcome!
<ul>
  {% for post in site.posts %}
    <li>
      <a href="blog/{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
