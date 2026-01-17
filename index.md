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


## CTF Writeups (under construction, will probably change to posts about specific writeups)
<ul>
  {% for writeup in site.ctf %}
    <li>
      <a href="{{ writeup.url }}">{{ writeup.event }}: {{ writeup.title }}</a> - {{ writeup.date | date: "%b %d, %Y" }}
      {% if writeup.tags %}
        <br><span style="font-size: 0.85em;">Focus: {{ writeup.tags | join: ", " }}</span>
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


