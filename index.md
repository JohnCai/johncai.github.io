---
layout: page
title: John Cai
tagline: 代码/敏捷/人
---
{% include JB/setup %}   

<ul class="posts">
  {% for post in site.posts %}
    <li>    	
    	<a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>    
    	<div> →_→ &nbsp;{{ post.description }} </div>
    	<br/>
    </li>
  {% endfor %}
</ul>