---
layout: default
---

{% include insights-dashboard.html %}

<h2 class="section-label">// published posts</h2>

<div class="post-grid">
{% for post in site.posts %}
  <a href="{{ post.url | relative_url }}" class="post-card">
    <span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
    <span class="post-title">{{ post.title }}</span>
    {% if post.tags.size > 0 %}
    <span class="post-tags">
      {% for tag in post.tags %}<span class="tag">{{ tag }}</span>{% endfor %}
    </span>
    {% endif %}
  </a>
{% endfor %}
</div>
