---
layout: page
permalink: /papers/
title: Papers
description: Academic writtings by categories in reversed chronological order.
years: [2023, 2020]
years_preprints: [2023, 2022]
years_technical: [2023, 2022]
nav: true
nav_order: 1
toc:
  sidebar: left
---
<!-- _pages/publications.md -->
<div class="publications">

<h1>Submitted articles &amp; preprints</h1>
{%- for y in page.years_preprints %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f preprint -q @*[year={{y}}]* %}
{% endfor %}

<h1>Journal articles</h1>
{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f journal -q @*[year={{y}}]* %}
{% endfor %}

<h1>Technical reports &amp; theses</h1>

{% bibliography -f technical %}

</div>
