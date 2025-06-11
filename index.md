---
layout: default
title: 首頁
---

# 筆記空間
這邊我會記錄在開發過程中遇到的一些狀況的處理方案

## 最新文章

{% for post in site.posts %}
  <h3><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h3>
  <p><small>{{ post.date | date: "%Y-%m-%d" }}</small></p>
  <p>{{ post.excerpt }}</p>
{% endfor %}
