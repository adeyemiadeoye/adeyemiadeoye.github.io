---
layout: page
permalink: /notes/
title: notes
description:  <em> Coming soon.</em> Some of my (digital handwritten) notes on topics I've been learning since the beginning of my PhD in 2020.
nav: false
nav_order: 4
display_categories: [convex optimization, system identification &amp; control, nonsmooth optimization, neural networks, nonlinear programming]
---

{%- for category in page.display_categories %}
<h2 class="category">{{ category }}</h2>
{%- assign categorized_notes = site.notes | where: "category", category -%}
<ul>
{%- for item in categorized_notes %}
    <li>  <strong>{{ item.title | remove: '<p>' | remove: '</p>' | emojify }}</strong><a href="{{ item.pdf | prepend: '/assets/pdf/' | relative_url }}" class="btn btn-sm z-depth-1" role="button" style="color:blue;" target="_blank">PDF</a><em>[{{ item.date | date: "%b %-d, %Y" }}]</em> </li>
{%- endfor %}
</ul>
{%- endfor %}