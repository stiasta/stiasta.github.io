---
layout: page
title: An Agile and Development Blog
---
<p>
  ... This is currently a work in progress. Luckily there are some blog posts already written. Go to town!
</p>
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


