---
layout: page
title: John Cai
tagline: 代码/敏捷/人
---
{% include JB/setup %}   

<h4> &nbsp; A wife calls her programmer husband and tells him, "While you're out, buy some milk." <small> He never returns home. </small></h4>

<br/>
<ul class="posts">
  {% for post in site.posts %}
    <li>    	
    	<a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a>    
    	<div> →_→ &nbsp;{{ post.description }} </div>
    	<br/>
    </li>
  {% endfor %}
</ul>