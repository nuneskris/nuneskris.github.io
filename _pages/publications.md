---
layout: archive
title: "the art and science of building data products"
permalink: /publications/
author_profile: false
---

{% include base_path %}

{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}
