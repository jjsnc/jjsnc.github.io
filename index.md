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


## CTFs
<ul>
  {% for ctf in site.data.ctfs %}
    <li>
      <strong>{{ ctf.name }} — {{ ctf.date | date: "%B %Y" }}</strong>
      {% if ctf.focus %}<br>Focus: {{ ctf.focus | join: ", " }}{% endif %}
    </li>
  {% endfor %}
</ul>

## Projects
