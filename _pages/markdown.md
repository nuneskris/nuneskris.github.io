---
layout: archive
title: "the art and science of building data products"
permalink: /markdown/
author_profile: true
---

Type something here
{% include base_path %}

{% for post in site.markdown reversed %}
  {% include archive-single.html %}
{% endfor %}
