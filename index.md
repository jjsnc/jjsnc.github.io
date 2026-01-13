---
layout: default
title: Home
---

# What have I been up to lately...

---

## Latest Blog Posts

<ul>
  {% for post in site.posts limit:10 %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> - {{ post.date | date: "%b %d, %Y" }}
    </li>
  {% endfor %}
</ul>


## CTFs (under construction, will probably change to posts about specific writeups)
<ul>
  {% for ctf in site.data.ctfs %}
    <li>
      <span style="color: #393939;">{{ ctf.name }}</span> - {{ ctf.date | date: "%b %d, %Y" }}
      {% if ctf.focus %}
        <br><span style="font-size: 0.85em;">Focus: {{ ctf.focus | join: ", " }}</span>
      {% endif %}
    </li>
  {% endfor %}
</ul>

## Projects

<ul>
  {% for project in site.data.projects %}
    <li>
      <a href="{{ project.url }}">{{ project.name }}</a> - {{ project.range }}
    </li>
  {% endfor %}
</ul>


