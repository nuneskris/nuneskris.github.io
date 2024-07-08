---
layout: archive
title: "Machine Learning"
permalink: /mypublications/
author_profile: true
---

{% include base_path %}

{% for post in site.mypublications reversed %}
  {% include archive-single.html %}
{% endfor %}
