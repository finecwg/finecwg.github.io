---
title: "Bioinformatics"
layout: archive
permalink: /bioinformatics
author_profile: true
sidebar:
    nav: "sidebar-category"
---


{% assign posts = site.categories.bioinformatics %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}