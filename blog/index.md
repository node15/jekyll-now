---
layout: default
title: Table of Contents
---

// template

{% for cat in site.category-list %}
### {{ cat }}
<ul>
{% for page in site.pages %}
{% if page.blog == true %}
{% for pc in page.categories %}
{% if pc == cat %}
<li>
  <a href="{{ page.url }}">{{ page.title }}</a> &mdash; {{ page.desc }}
</li>
{% endif %} <!-- cat-match-p -->
{% endfor %} <!-- page-category -->
{% endif %} <!-- blog-p -->
{% endfor %} <!-- page -->
</ul>
{% endfor %} <!-- cat --> 
