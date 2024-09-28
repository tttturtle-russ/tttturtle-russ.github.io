---
permalink: /posts
title: "Posts"
excerpt: ""
author_profile: true
redirect_from: 
  - /posts
---

{% if site.google_scholar_stats_use_cdn %}
{% assign gsDataBaseUrl = "https://cdn.jsdelivr.net/gh/" | append: site.repository | append: "@" %}
{% else %}
{% assign gsDataBaseUrl = "https://raw.githubusercontent.com/" | append: site.repository | append: "/" %}
{% endif %}
{% assign url = gsDataBaseUrl | append: "google-scholar-stats/gs_data_shieldsio.json" %}

- [KCOV: 用于模糊测试的代码覆盖率]("/kcov.md)