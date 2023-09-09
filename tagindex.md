---
layout: page
title: Tag Index
permalink: /tagindex/
---

<html>
  <head>
    <title>Tag Index</title>
  </head>
    <body>
      <div>
        {% assign sorted_tags = site.tags | sort %} {% for tag in sorted_tags %} {% assign tagName = tag | first | downcase %} {% assign postsCount = tag | last | size %} <br><a href='/tag/{{ tagName }}'><i class='glyphicon glyphicon-tag'></i>{{ tagName }}</a>({{ postsCount }})</br> {% endfor %}
      </div>
    </body>
</html>

