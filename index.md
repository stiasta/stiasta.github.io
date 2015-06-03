---
layout: page
title: An Agile and Development Blog
---
<p>
  ... This is currently a work in progress. Luckily there are some blog posts already written. Go to town!
</p>

<div class="container">
    <article>
        <div class="row">
            <header>
                <h3>Posts</h3>
            </header>
        </div>
        <section>
            <!--<div class="row">
                <label class="hidden-xs col-xs-10 col-sm-11 col-md-11">
                    Keywords
                </label>
            </div>-->
            <div class="row">
                <div class="list-group col-xs-10 col-sm-11 col-md-11">
                    {% for post in site.posts %}
                    <a class="list-group-item"
                       href="{{ post.url }}"> 
                        <i>{{ post.date | date_to_string }}</i> - {{ post.title }}

                        <i class="hidden-xs pull-right">
                            {{ post.categories | array_to_sentence_string }}
                        </i>
                    </a>
                    {% endfor %}
                </div>
            </div>
        </section>
    </article>
</div>


