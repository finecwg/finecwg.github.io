---
title: "Medical Informatics"
layout: archive
permalink: /medical-informatics
author_profile: true
sidebar:
    nav: "sidebar-category"
---


{% assign posts = site.categories.medical-informatics %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}