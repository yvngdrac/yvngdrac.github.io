---
layout: default
title: My Writeups
---

# Welcome to My Writeups

This is where I share my security research, CTF writeups, and technical posts.

## Latest Posts

{% for post in site.posts %}
  ### [{{ post.title }}]({{ post.url }})
  *{{ post.date | date_to_long_string }}*
  
{% endfor %}

---

*Happy reading! 🔒*
