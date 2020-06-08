---
layout: page
title: About
description: 打码改变世界
keywords: John Cai, 蔡建斌
comments: true
menu: 关于
permalink: /about/
---

我是蔡建斌。

分享敏捷軟件研發百態。

永遠在通往「出來拔菜」的路上奔跑。

## 联系

<ul>
{% for website in site.data.social %}
<li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
{% endfor %}

<li>
微信公众号：<br />
<img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ assets_base_url }}/assets/images/qrcode.jpg" alt="蔡建斌" />
</li>

</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
