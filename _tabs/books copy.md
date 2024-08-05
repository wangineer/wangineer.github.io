---
layout: home
icon: fas fa-book
title      : "Books"
date       : "2023-02-26"
author     : Kevin
categories : ['Movie']
order: 7
---
{% for post in site.posts %}
  {% if post.categories contains 'Movie' %} 
#### [{{post.title}}]({{post.url}}) - <small>{{ post.date | date: "%B %-d %Y" }}</small><br/> - {{post.description}}
  {% endif %}
{% endfor %}