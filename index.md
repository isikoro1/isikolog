---
layout: default
title: Home
---

<section class="home-intro">
  <h2 class="home-title">Posts</h2>
</section>

<ol class="post-feed">
  {% for post in site.posts %}
    {% assign excerpt_source = post.excerpt | default: post.content %}
    {% assign clean_excerpt = excerpt_source | markdownify | strip_html | strip_newlines | replace: "  ", " " | strip %}
    <li class="post-feed-item">
      <a class="post-card-link" href="{{ post.url | relative_url }}">
        <p class="post-card-meta">{{ post.date | date: "%Y-%m-%d" }}</p>
        <h3 class="post-card-title">{{ post.title }}</h3>
        {% if clean_excerpt != "" %}
          <p class="post-card-excerpt">{{ clean_excerpt }}</p>
        {% endif %}
        {% if post.tags and post.tags.size > 0 %}
          <div class="post-card-tags" aria-label="Tags">
            {% for tag in post.tags %}
              <span class="tag-chip">{{ tag }}</span>
            {% endfor %}
          </div>
        {% endif %}
      </a>
    </li>
  {% endfor %}
</ol>

<footer class="home-footer">
  <a href="https://github.com/isikoro1/isikolog?tab=readme-ov-file">GitHub</a>
  <a href="https://qiita.com/isikoro">Qiita</a>
  <a href="https://isikoro1.github.io/isikoro1site/">isikoro1Site</a>
  <a href="https://x.com/mot1173137">X</a>
</footer>
