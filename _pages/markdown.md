---
layout: archive
title: "the art and science of building data products"
permalink: /markdown/
author_profile: true
---

{% include base_path %}

{% for post in site.markdowns reversed %}
  {% include archive-single.html %}
{% endfor %}
