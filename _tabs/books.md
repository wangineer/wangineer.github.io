---
layout: post
icon: fas fa-stream
title      : "Book"
fid        : 50 
date       : "2023-02-26"
author     : Kevin
categories : ['Linux']
order: 6
---
{% for post in site.posts %}
  {% if post.categories contains 'Linux' %}
    
#### [{{post.title}}]({{post.url}})<br/><small>{{ post.date | date_to_long_string }}</small>
By: {{post.author}}
    
  {% endif %}
{% endfor %}