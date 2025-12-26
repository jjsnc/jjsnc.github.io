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
      <strong>{{ ctf.name }}</strong> — {{ ctf.date | date: "%B %Y" }}
      {% if ctf.focus %}<br><em>Focus:</em> {{ ctf.focus | join: ", " }}{% endif %}
    </li>
  {% endfor %}
</ul>

## Projects
