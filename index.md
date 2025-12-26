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
      <span style="color:black;">{{ ctf.name }}</span> - {{ ctf.date | date: "%b %d, %Y" }}
      {% if ctf.focus %}
        <br><span style="font-size: 0.85em;">Focus: {{ ctf.focus | join: ", " }}</span>
      {% endif %}
    </li>
  {% endfor %}
</ul>

## Projects
