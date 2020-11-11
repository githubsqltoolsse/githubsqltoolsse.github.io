---
layout: posts
title: Post by Category
permalink: /categoryview/
sitemap: false
---

<div id="navtest">
<ul>
  {% for item in include.nav %}
    <li><a href="{{ item.url }}">{{ item.title }}</a></li>

    {% if item.subnav %}
      {% include nav.html nav=item.subnav %}
    {% endif %}
  {% endfor %}
</ul>
  </div>