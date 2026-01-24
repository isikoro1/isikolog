---
layout: default
title: Tags
---

# Tags

{% assign tags = site.tags | sort %}
<ul>
{% for tag in tags %}
  <li>
    <strong>{{ tag[0] }}</strong> ({{ tag[1].size }})
    <ul>
    {% for post in tag[1] %}
      <li><a href="{{ post.url | relative_url }}">{{ post.title }}</a></li>
    {% endfor %}
    </ul>
  </li>
{% endfor %}
</ul>
