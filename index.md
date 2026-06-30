---
layout: default
title: Home
---

<section class="home-intro">
  <h2 class="home-title">Posts</h2>
  <p class="home-copy">日々のメモ、作ったもの、学んだことを残しています。</p>
</section>

{% assign has_games = false %}
{% assign has_tools = false %}
{% for post in site.posts %}
  {% assign categories_text = post.categories | join: "," %}
  {% if post.kind == "game" or post.kind == "ゲーム" or post.category == "game" or post.category == "ゲーム" or categories_text contains "game" or categories_text contains "ゲーム" %}
    {% assign has_games = true %}
  {% endif %}
  {% if post.kind == "tool" or post.kind == "ツール" or post.category == "tool" or post.category == "ツール" or categories_text contains "tool" or categories_text contains "ツール" %}
    {% assign has_tools = true %}
  {% endif %}
{% endfor %}

{% if has_games %}
  <section class="feed-section">
    <h3 class="feed-section-title">Games</h3>
    <ol class="post-feed">
      {% for post in site.posts %}
        {% assign categories_text = post.categories | join: "," %}
        {% if post.kind == "game" or post.kind == "ゲーム" or post.category == "game" or post.category == "ゲーム" or categories_text contains "game" or categories_text contains "ゲーム" %}
          {% assign excerpt_source = post.excerpt | default: post.content %}
          {% assign clean_excerpt = excerpt_source | markdownify | strip_html | strip_newlines | replace: "  ", " " | strip %}
          {% assign post_kind = post.kind | default: post.category | default: "game" %}
          <li class="post-feed-item">
            <a class="post-card-link" href="{{ post.url | relative_url }}">
              <div class="post-card-topline">
                <p class="post-card-meta">{{ post.date | date: "%Y-%m-%d" }}</p>
                <span class="post-kind-badge">{{ post_kind }}</span>
              </div>
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
        {% endif %}
      {% endfor %}
    </ol>
  </section>
{% endif %}

{% if has_tools %}
  <section class="feed-section">
    <h3 class="feed-section-title">Tools</h3>
    <ol class="post-feed">
      {% for post in site.posts %}
        {% assign categories_text = post.categories | join: "," %}
        {% if post.kind == "tool" or post.kind == "ツール" or post.category == "tool" or post.category == "ツール" or categories_text contains "tool" or categories_text contains "ツール" %}
          {% assign excerpt_source = post.excerpt | default: post.content %}
          {% assign clean_excerpt = excerpt_source | markdownify | strip_html | strip_newlines | replace: "  ", " " | strip %}
          {% assign post_kind = post.kind | default: post.category | default: "tool" %}
          <li class="post-feed-item">
            <a class="post-card-link" href="{{ post.url | relative_url }}">
              <div class="post-card-topline">
                <p class="post-card-meta">{{ post.date | date: "%Y-%m-%d" }}</p>
                <span class="post-kind-badge">{{ post_kind }}</span>
              </div>
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
        {% endif %}
      {% endfor %}
    </ol>
  </section>
{% endif %}

<section class="feed-section">
  <h3 class="feed-section-title">Articles</h3>
  <ol class="post-feed">
    {% for post in site.posts %}
      {% assign categories_text = post.categories | join: "," %}
      {% unless post.kind == "game" or post.kind == "ゲーム" or post.kind == "tool" or post.kind == "ツール" or post.category == "game" or post.category == "ゲーム" or post.category == "tool" or post.category == "ツール" or categories_text contains "game" or categories_text contains "ゲーム" or categories_text contains "tool" or categories_text contains "ツール" %}
        {% assign excerpt_source = post.excerpt | default: post.content %}
        {% assign clean_excerpt = excerpt_source | markdownify | strip_html | strip_newlines | replace: "  ", " " | strip %}
        {% assign post_kind = post.kind | default: post.category | default: "post" %}
        <li class="post-feed-item">
          <a class="post-card-link" href="{{ post.url | relative_url }}">
            <div class="post-card-topline">
              <p class="post-card-meta">{{ post.date | date: "%Y-%m-%d" }}</p>
              <span class="post-kind-badge">{{ post_kind }}</span>
            </div>
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
      {% endunless %}
    {% endfor %}
  </ol>
</section>

<footer class="home-footer">
  <a href="https://github.com/isikoro1/isikolog?tab=readme-ov-file">GitHub</a>
  <a href="https://qiita.com/isikoro">Qiita</a>
  <a href="https://isikoro1.github.io/isikoro1site/">isikoro1Site</a>
  <a href="https://x.com/mot1173137">X</a>
</footer>
