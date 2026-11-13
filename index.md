---
layout: default
title: Joshua Hia
---

# Joshua Hia — Data Science Portfolio

*Data scientist · Causal inference · Economics · Tech*

{% assign sorted_writeups = site.writeups | sort: "order" %}
{% for writeup in sorted_writeups %}
## [{{ writeup.title }}]({{ writeup.url | relative_url }})

{{ writeup.summary }}

{% if writeup.image %}
![]({{ writeup.image | relative_url }})
{% endif %}

{% endfor %}
