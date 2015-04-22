---
layout: page
title: "Pagestest"
description: ""
---
{% include JB/setup %}

<ul class="pages">
  {% for page in page.pages %}
    <li><a href="{{ BASE_PATH }}{{ page.url }}">{{ page.title }}</a></li>
  {% endfor %}
</ul> 
