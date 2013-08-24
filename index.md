---
layout: page
title: John Cai
tagline: 洼则盈
---
{% include JB/setup %}
    
## 我的博客

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>