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
      </div>
      <script> 
        var mytable = "<table cellpadding=\"0\" cellspacing=\"0\"><tbody><tr>";
        {% assign sorted_tags = site.tags | sort %} {% for tag in sorted_tags %} {% assign tagName = tag | first | downcase %}{% assign postsCount = tag | last | size %} 
          mytable += "<td>[" + "<a href='/tag/{{ tagName }}'><i class='glyphicon glyphicon-tag'></i>{{ tagName }}</a>({{ postsCount }})" +"]</td>" {% endfor %}
          mytable += "</tr></tbody></table>";
          document.write(mytable);
      </script>
    </body>
</html>

