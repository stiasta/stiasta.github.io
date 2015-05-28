---
layout: page
title: An Agile and Development Blog
---
<p>
  ... This is currently a work in progress. Luckily there are some blog posts already written. Go to town!
</p>

{% for post in site.posts %}
<article class="panel panel-default">
    <header class="panel-heading">
        <a href="{{ post.url }}"> 
            <h3 class="panel-title">{{ post.title }}</h3>
        </a>
    </header>
    <section>
        {{ post.content }}
    </section>
    <footer class="panel-footer">
        <i>{{ post.date | date_to_string }}</i>
    </footer>
</article>
{% endfor %}


