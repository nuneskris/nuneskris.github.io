---
layout: archive
title: "the art and science of building data products"
permalink: /publications/
author_profile: true
---
In the foundation section, I defined a high-level aspirational vision of the data analytics capabilities required to deliver business value. Now, we need frameworks and solution concepts that can consistently deliver these capabilities. It is important to consider various factors that influence how data analytics solutions are designed and implemented.

The architecture needs to consider each of these elements
* ***assumptions*** - These are conditions believed to be true for the architecture but have not yet been validated.
* ***constraints*** - These are limitations that must be adhered to, such as budget, time, technology, or legal restrictions.
* ***domain-specific principles*** - These are guiding principles specific to the domain that influence how the architecture is developed.
* ***policies & standards*** - These are established norms, specifications, and rules that the architecture must comply with.

{% include base_path %}

{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}
