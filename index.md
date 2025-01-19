# Welcome to My Blog

Here are all of my posts:

{% for post in site.posts %}
  <div class="post-preview">
    <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    <p><em>{{ post.date | date: "%B %d, %Y" }}</em></p>
    <p>{{ post.excerpt }}</p>
    <a href="{{ post.url | relative_url }}">Read more</a>
  </div>
{% endfor %}
