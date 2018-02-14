---
layout: default
---
{% assign tags = site.tags | sort %}
{% for tag in tags %}
## <i class='fas fa-tag'></i> {{ tag[0] }}
{% for page in tag[1] %}
  [{{ page.title}}]({{ page.url }})
{% endfor %}
{% endfor %}
