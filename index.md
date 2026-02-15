---
layout: default
title: Home
---

## Posts
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.date | date: "%Y-%m-%d" }} - {{ post.title }}</a>
    </li>
  {% endfor %}
</ul>

<footer>
  <div>
    <a href= "https://github.com/isikoro1/isikolog?tab=readme-ov-file">
      Github
    </a>
  </div>
</footer>
