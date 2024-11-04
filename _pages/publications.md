---
layout: page
permalink: /papers/
title: Papers
description: Academic writtings by categories in reversed chronological order.
years: [2024, 2023, 2020]
years_preprints: [2024, 2023]
years_technical: [2023, 2022]
nav: false
nav_order: 2
# toc:
#   sidebar: left
---
<!-- _pages/publications.md -->
<div class="publications">

<h1>Preprints</h1>
<br><br>
<h6><i>[Some may be under review; feedback is welcome before publication.]</i></h6>
{%- for y in page.years_preprints %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f preprint -q @*[year={{y}}]* %}
{% endfor %}

<h1>Journal articles</h1>
{%- for y in page.years %}
  <h2 class="year">{{y}}</h2>
  {% bibliography -f journal -q @*[year={{y}}]* %}
{% endfor %}

<h1 style="margin-bottom: 0.5em;">Technical reports &amp; theses</h1>

{% bibliography -f technical %}

</div>
