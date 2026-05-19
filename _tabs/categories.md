---
layout: default
title: Categories
permalink: /categories/
---

<div class="page-content">
  <h1 class="page-title">Categories</h1>
  
  {% assign categories = site.categories | sort %}
  {% for category in categories %}
    {% assign category_name = category[0] %}
    {% assign posts_count = category[1] | size %}
    
    <div class="category-group">
      <h2 id="{{ category_name | slugify }}" class="category-heading">
        <i class="fas fa-folder-open"></i> {{ category_name }}
        <span class="post-count">{{ posts_count }}</span>
      </h2>
      
      <ul class="category-posts">
        {% for post in category[1] %}
          <li>
            <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
            <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span>
          </li>
        {% endfor %}
      </ul>
    </div>
  {% endfor %}
</div>

<style>
.page-content {
  max-width: 800px;
  margin: 0 auto;
}

.page-title {
  font-family: var(--mono);
  font-size: 2rem;
  color: var(--text-head);
  margin-bottom: 2rem;
  border-bottom: 2px solid var(--accent);
  display: inline-block;
  padding-bottom: 0.5rem;
}

.category-group {
  margin-bottom: 2rem;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 8px;
  overflow: hidden;
}

.category-heading {
  font-family: var(--mono);
  font-size: 1.2rem;
  padding: 1rem 1.2rem;
  background: var(--bg-sidebar);
  border-bottom: 1px solid var(--border);
  margin: 0;
}

.category-heading i {
  color: var(--accent);
  margin-right: 0.5rem;
}

.post-count {
  font-size: 0.8rem;
  color: var(--text-muted);
  margin-left: 0.8rem;
  font-weight: normal;
}

.category-posts {
  list-style: none;
  padding: 0;
  margin: 0;
}

.category-posts li {
  padding: 0.8rem 1.2rem;
  border-bottom: 1px solid var(--border);
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.category-posts li:last-child {
  border-bottom: none;
}

.category-posts li a {
  color: var(--text);
  font-size: 0.9rem;
  transition: color 0.15s;
}

.category-posts li a:hover {
  color: var(--accent);
}

.category-posts .post-date {
  font-size: 0.75rem;
  color: var(--text-muted);
  font-family: var(--mono);
}

@media (max-width: 600px) {
  .category-posts li {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>
