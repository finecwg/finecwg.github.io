---
title: "📖 Books"
layout: archive
permalink: /books
author_profile: true
sidebar:
    nav: "books"
---


{% assign posts = site.categories.books %}
{% for post in posts %} {% include archive-single.html type=page.entries_layout %} {% endfor %}