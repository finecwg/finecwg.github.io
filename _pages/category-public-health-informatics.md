---
title: "Public Health Informatics"
layout: archive
permalink: /public-health-informatics
author_profile: true
sidebar:
    nav: "sidebar-category"
---


{% assign posts = site.categories.public-health-informatics %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}