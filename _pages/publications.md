---
layout: archive
title: "the art and science of building data products"
permalink: /publications/
author_profile: true
---
In the foundation section I defined a high-level aspirational vision of data analytics capabilities required to deliver business value. Now we need frameworks and solution concepts which can consistently deliver these capabilities.It is important to consider various factors that influence how data analytics solutions are designed and implemented. 

They need to consider esch of these elements 
* ***assumptions*** - the conditions that are believed to be true for the architecture but have not been validated
* ***constraints*** - limitations that must be adhered to, such as budget, time, technology, or legal restrictions
* ***domain-specific principles*** - guiding principles specific that influence how architecture
* ***policies & standards*** - established norms,  specifications & rules that the architecture must comply with

{% include base_path %}

{% for post in site.publications reversed %}
  {% include archive-single.html %}
{% endfor %}
