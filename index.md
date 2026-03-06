---
layout: default
title: SOC Reports
---

# SOC Incident Reports

Welcome to my SOC investigation reports.

## Latest Reports

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
