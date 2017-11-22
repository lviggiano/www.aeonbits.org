---
layout: page
title: Welcome to aeonbits.org 
tagline: (my open source contribution)
---
{% include JB/setup %}

Have a look at [OWNER API](http://owner.aeonbits.org), a simple API to ease Java Properties file handling.
    
### Latest posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
