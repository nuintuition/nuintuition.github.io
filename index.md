---
layout: default
title: 首頁
---

# 隨意寫寫
隨手紀錄程式開發所碰到的大小事

## 最新文章

{% for post in site.posts %}
  <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <p><small>{{ post.date | date: "%Y-%m-%d" }}</small></p>
  <p>{{ post.excerpt }}</p>
{% endfor %}
