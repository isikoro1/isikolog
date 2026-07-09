---
layout: default
title: Home
---

<section class="home-intro">
  <h2 class="home-title">Posts</h2>
  <p class="home-copy">日々のメモ、作ったもの、学んだことを残しています。</p>
</section>

<div class="post-tabs">
  <input class="tab-radio" type="radio" name="post-tab" id="tab-tech" checked>
  <input class="tab-radio" type="radio" name="post-tab" id="tab-idea">

  <div class="post-search">
    <label class="post-search-label" for="post-search-input">Search posts</label>
    <input
      class="post-search-input"
      id="post-search-input"
      type="search"
      placeholder="Search title, summary, tags"
      autocomplete="off"
      spellcheck="false"
    >
  </div>

  <div class="tab-list" role="tablist" aria-label="Post categories">
    <label class="tab-button tab-button-tech" for="tab-tech" role="tab">Tech</label>
    <label class="tab-button tab-button-idea" for="tab-idea" role="tab">Idea</label>
  </div>

  <div class="tab-panels">
    <section class="feed-section tab-panel tab-panel-tech" aria-label="Tech posts">
      <h3 class="feed-section-title">Tech</h3>
      <ol class="post-feed">
        {% for post in site.posts %}
          {% assign post_category = post.section | default: "idea" %}
          {% if post_category == "tech" %}
            {% assign excerpt_source = post.excerpt | default: post.content %}
            {% assign clean_excerpt = excerpt_source | markdownify | strip_html | strip_newlines | replace: "  ", " " | strip %}
            {% assign post_kind = post.kind | default: "Tech" %}
            {% assign tags_text = post.tags | join: " " %}
            {% assign search_text = post.title | append: " " | append: clean_excerpt | append: " " | append: tags_text | downcase %}
            <li class="post-feed-item" data-search="{{ search_text | escape }}">
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
      <p class="post-no-results" hidden aria-live="polite">No matching Tech posts.</p>
    </section>

    <section class="feed-section tab-panel tab-panel-idea" aria-label="Idea posts">
      <h3 class="feed-section-title">Idea</h3>
      <ol class="post-feed">
        {% for post in site.posts %}
          {% assign post_category = post.section | default: "idea" %}
          {% if post_category == "idea" %}
            {% assign excerpt_source = post.excerpt | default: post.content %}
            {% assign clean_excerpt = excerpt_source | markdownify | strip_html | strip_newlines | replace: "  ", " " | strip %}
            {% assign post_kind = post.kind | default: "Idea" %}
            {% assign tags_text = post.tags | join: " " %}
            {% assign search_text = post.title | append: " " | append: clean_excerpt | append: " " | append: tags_text | downcase %}
            <li class="post-feed-item" data-search="{{ search_text | escape }}">
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
      <p class="post-no-results" hidden aria-live="polite">No matching Idea posts.</p>
    </section>
  </div>
</div>

<footer class="home-footer">
  <a href="https://github.com/isikoro1/isikolog?tab=readme-ov-file">GitHub</a>
  <a href="https://qiita.com/isikoro">Qiita</a>
  <a href="https://isikoro.dev/">App</a>
  <a href="https://x.com/mot1173137">X</a>
</footer>

<script>
  const postSearchInput = document.querySelector("#post-search-input");

  if (postSearchInput) {
    const url = new URL(window.location.href);
    const initialQuery = url.searchParams.get("q");
    if (initialQuery) postSearchInput.value = initialQuery;

    const filterPosts = () => {
      const query = postSearchInput.value.trim().toLowerCase();

      document.querySelectorAll(".tab-panel").forEach((panel) => {
        let visibleCount = 0;

        panel.querySelectorAll(".post-feed-item").forEach((item) => {
          const isVisible = item.dataset.search.includes(query);
          item.hidden = !isVisible;
          if (isVisible) visibleCount += 1;
        });

        const noResults = panel.querySelector(".post-no-results");
        if (noResults) noResults.hidden = visibleCount > 0;
      });

      if (query) {
        url.searchParams.set("q", query);
      } else {
        url.searchParams.delete("q");
      }

      window.history.replaceState({}, "", url);
    };

    postSearchInput.addEventListener("input", filterPosts);
    filterPosts();
  }
</script>
