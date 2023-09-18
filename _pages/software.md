---
layout: page
title: software
permalink: /software/
description: 
nav: true
nav_order: 3
display_categories: 
horizontal: true
---

<div class="software">
{%- if site.enable_project_categories and page.display_categories %}
  <!-- Display categorized software -->
  {%- for category in page.display_categories %}
  <h2 class="category">{{ category }}</h2>
  {%- assign categorized_software = site.software | where: "category", category -%}
  {%- assign sorted_software = categorized_software | sort: "importance" %}
  <!-- Generate cards for each software -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for software in sorted_software -%}
      {% include software_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for software in sorted_software -%}
      {% include software.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
  {% endfor %}

{%- else -%}
<!-- Display software without categories -->
  {%- assign sorted_software = site.software | sort: "importance" -%}
  <!-- Generate cards for each software -->
  {% if page.horizontal -%}
  <div class="container">
    <div class="row row-cols-2">
    {%- for software in sorted_software -%}
      {% include software_horizontal.html %}
    {%- endfor %}
    </div>
  </div>
  {%- else -%}
  <div class="grid">
    {%- for software in sorted_software -%}
      {% include software.html %}
    {%- endfor %}
  </div>
  {%- endif -%}
{%- endif -%}
</div>

<!-- <ul>
<li>
<a style="text-decoration:underline" href="https://github.com/adeyemiadeoye/SelfConcordantSmoothOptimization.jl" target="_blank">SelfConcordantSmoothOptimization.jl</a> <p> A Julia package implementing self-concordant smoothing algorithms for convex composite optimization introduced in <a style="text-decoration:none" href="https://arxiv.org/abs/2309.01781" target="_blank">this paper</a>.
</p>
</li>
<br>
</ul> -->