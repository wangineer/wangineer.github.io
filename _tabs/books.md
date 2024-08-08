---
layout: books
icon: fas fa-book
title      : "Books"
fid        : 50 
date       : "2023-02-26"
author     : Kevin
order: 6
---
{% for post in site.posts %}
  {% if post.categories contains 'Linux' %} 

  {% endif %}
{% endfor %}

