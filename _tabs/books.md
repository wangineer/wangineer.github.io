---
layout: post
icon: fas fa-stream
title      : "Book Recommendations"
fid        : 50 
date       : "2023-02-26"
categories : ['Book']
order: 6
---
{% for post in site.posts %}
  {% if post.categories contains 'Linux' %}
    
#### [{{post.title}}]({{post.url}})<br/><small>{{ post.date | date_to_long_string }}</small>
By: {{post.author}}
    
  {% endif %}
{% endfor %}