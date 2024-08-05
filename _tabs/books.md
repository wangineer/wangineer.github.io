---
layout: page
icon: fa-book
title      : "Books"
fid        : 50 
date       : "2023-02-26"
author     : Kevin
categories : ['Linux']
order: 6
---
{% for post in site.posts %}
  {% if post.categories contains 'Linux' %}
    
#### [{{post.title}}]({{post.url}}) - {{ post.date | date: "%B %-d %Y" }}<br/>{{post.description}}<br/><small>{{ post.date | date: "%B %-d %Y" }}</small>
    
  {% endif %}
{% endfor %}
