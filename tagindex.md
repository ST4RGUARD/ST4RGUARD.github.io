---
layout: page
title: Tag Index
permalink: /tagindex/
---
<!doctype html>
<html>
  <head>
    <title>Tag Index</title>
  </head>
    <body>
      <div>
        {% assign sorted_tags = site.tags | sort %} {% for tag in sorted_tags %} {% assign tagName = tag | first | downcase %} {% assign postsCount = tag | last | size %} <a href='/tag/{{ tagName }}'><i class='glyphicon glyphicon-tag'></i>{{ tagName }}</a>({{ postsCount }}) {% endfor %}
      </div>
      <script> 
        var mytable = "<table cellpadding=\"0\" cellspacing=\"0\"><tbody><tr>";
        mytable += "</tr><tr>";
        mytable += "<td>[" + tag + "]</td>";
        mytable += "</tr></tbody></table>";
      </script>
    </body>
</html>

