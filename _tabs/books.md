---
layout: page
icon: fas fa-stream
title      : "Books"
fid        : 50 
date       : "2023-02-26"
author     : Kevin
categories : ['Linux']
order: 6
---
{% for post in site.posts %}
  {% if post.categories contains 'Linux' %}
    
#### [{{post.title}}]({{post.url}})<br/><small>{{ post.date | date: "%B %-d %Y" }}<br/>By:</small><br/>{{post.author}}
By: {{post.author}}
    
  {% endif %}
{% endfor %}
