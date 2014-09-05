---
layout: page
title: Archive
permalink: /archive.html
---

{% for post in site.posts %}
  {% capture postmonth %}{{ post.date | date: "%B" }}{% endcapture %}
  {% capture postyear %}{{ post.date | date: "%Y" }}{% endcapture %}
  {% if postmonth != month or postyear != year %}
    {% assign month = postmonth %}
    {% assign year = postyear %}
    {% if forloop.first %}
    {% else %}
     
    {% endif %}
### {{ month }} {{ year }}  
  {% endif %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}
