---
layout: default
---

{% for post in site.posts %}
{% unless post.draft %}
<article class="post-preview">
  <h2><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
  <time>{{ post.date | date: "%B %d, %Y" }}</time>
  <p>{{ post.excerpt | strip_html | truncatewords: 40 }}</p>
</article>
{% endunless %}
{% endfor %}
