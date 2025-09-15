---
layout: default
title: Home
---

# Welcome to My Cybersecurity Journal

---

## Latest Blog Posts

<ul>
  {% for post in site.posts limit:10 %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%b %d, %Y" }}
    </li>
  {% endfor %}
</ul>


## Projects
