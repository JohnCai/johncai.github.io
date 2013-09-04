---
layout: page
title: A wife calls her programmer husband and tells him, "While you're out, buy some milk."
tagline: He never returns home.
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