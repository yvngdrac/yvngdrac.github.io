---
layout: default
title: Archives
permalink: /archives/
---

<div class="page-content">
  <h1 class="page-title">Archives</h1>
  
  {% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
  
  {% for year_group in posts_by_year %}
    <div class="archive-year">
      <h2 class="year-heading">
        <i class="fas fa-calendar-alt"></i> {{ year_group.name }}
        <span class="post-count">{{ year_group.items | size }} posts</span>
      </h2>
      
      {% assign posts_by_month = year_group.items | group_by_exp: "post", "post.date | date: '%B'" %}
      
      {% for month_group in posts_by_month %}
        <div class="archive-month">
          <h3 class="month-heading">{{ month_group.name }}</h3>
          <ul class="archive-posts">
            {% for post in month_group.items %}
              <li>
                <span class="post-day">{{ post.date | date: "%d" }}</span>
                <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
                <span class="post-categories">
                  {% for category in post.categories limit: 1 %}
                    <i class="fas fa-folder"></i> {{ category }}
                  {% endfor %}
                </span>
              </li>
            {% endfor %}
          </ul>
        </div>
      {% endfor %}
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

.archive-year {
  margin-bottom: 2rem;
  background: var(--bg-card);
  border: 1px solid var(--border);
  border-radius: 8px;
  overflow: hidden;
}

.year-heading {
  font-family: var(--mono);
  font-size: 1.2rem;
  padding: 1rem 1.2rem;
  background: var(--bg-sidebar);
  border-bottom: 1px solid var(--border);
  margin: 0;
}

.year-heading i {
  color: var(--accent);
  margin-right: 0.5rem;
}

.year-heading .post-count {
  font-size: 0.8rem;
  color: var(--text-muted);
  margin-left: 0.8rem;
  font-weight: normal;
}

.archive-month {
  border-bottom: 1px solid var(--border);
}

.archive-month:last-child {
  border-bottom: none;
}

.month-heading {
  font-family: var(--mono);
  font-size: 0.9rem;
  padding: 0.6rem 1.2rem;
  margin: 0;
  background: var(--bg);
  color: var(--accent);
}

.archive-posts {
  list-style: none;
  padding: 0;
  margin: 0;
}

.archive-posts li {
  padding: 0.6rem 1.2rem;
  border-bottom: 1px solid var(--border);
  display: flex;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.archive-posts li:last-child {
  border-bottom: none;
}

.post-day {
  font-family: var(--mono);
  font-size: 0.8rem;
  font-weight: 700;
  color: var(--accent);
  min-width: 35px;
}

.archive-posts li a {
  flex: 1;
  color: var(--text);
  font-size: 0.9rem;
  transition: color 0.15s;
}

.archive-posts li a:hover {
  color: var(--accent);
}

.post-categories {
  font-size: 0.7rem;
  color: var(--text-muted);
  font-family: var(--mono);
}

.post-categories i {
  margin-right: 0.2rem;
}

@media (max-width: 600px) {
  .archive-posts li {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.3rem;
  }
  
  .post-day {
    font-size: 0.7rem;
  }
}
</style>
