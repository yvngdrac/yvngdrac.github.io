---
layout: default
title: Tags
permalink: /tags/
---

<div class="page-content">
  <h1 class="page-title">Tags</h1>
  
  <div class="tags-cloud">
    {% assign tags = site.tags | sort %}
    {% for tag in tags %}
      {% assign tag_name = tag[0] %}
      {% assign posts_count = tag[1] | size %}
      <a href="#{{ tag_name | slugify }}" class="tag-link" data-count="{{ posts_count }}">
        {{ tag_name }}
      </a>
    {% endfor %}
  </div>

  <div class="tags-list">
    {% for tag in tags %}
      {% assign tag_name = tag[0] %}
      <div class="tag-section">
        <h2 id="{{ tag_name | slugify }}" class="tag-heading">
          <i class="fas fa-tag"></i> {{ tag_name }}
          <span class="post-count">{{ tag[1] | size }} posts</span>
        </h2>
        
        <ul class="tag-posts">
          {% for post in tag[1] %}
            <li>
              <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
              <span class="post-date">{{ post.date | date: "%B %d, %Y" }}</span>
            </li>
          {% endfor %}
        </ul>
      </div>
    {% endfor %}
  </div>
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

.tags-cloud {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 2rem;
  display: flex;
  flex-wrap: wrap;
  gap: 0.8rem;
}

.tag-link {
  font-family: var(--mono);
  font-size: 0.9rem;
  padding: 0.3rem 0.8rem;
  background: var(--tag-bg);
  border: 1px solid var(--tag-border);
  border-radius: 20px;
  color: var(--text);
  transition: all 0.15s;
  position: relative;
}

.tag-link:hover {
  color: var(--accent);
  border-color: var(--accent);
  transform: translateY(-2px);
}

.tag-link::after {
  content: attr(data-count);
  font-size: 0.7rem;
  margin-left: 0.3rem;
  color: var(--text-muted);
}

.tag-section {
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 8px;
  margin-bottom: 1.5rem;
  overflow: hidden;
}

.tag-heading {
  font-family: var(--mono);
  font-size: 1.1rem;
  padding: 0.8rem 1.2rem;
  background: var(--bg-sidebar);
  border-bottom: 1px solid var(--border);
  margin: 0;
}

.tag-heading i {
  color: var(--accent);
  margin-right: 0.5rem;
}

.tag-heading .post-count {
  font-size: 0.75rem;
  color: var(--text-muted);
  margin-left: 0.8rem;
  font-weight: normal;
}

.tag-posts {
  list-style: none;
  padding: 0;
  margin: 0;
}

.tag-posts li {
  padding: 0.7rem 1.2rem;
  border-bottom: 1px solid var(--border);
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tag-posts li:last-child {
  border-bottom: none;
}

.tag-posts li a {
  color: var(--text);
  font-size: 0.9rem;
  transition: color 0.15s;
}

.tag-posts li a:hover {
  color: var(--accent);
}

.tag-posts .post-date {
  font-size: 0.75rem;
  color: var(--text-muted);
  font-family: var(--mono);
}

@media (max-width: 600px) {
  .tag-posts li {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>
