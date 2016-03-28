---
layout: page
title: Portfolio
---

{% for repository in site.github.public_repositories %}
  <div class="portfolio-item">
  	<h2><a href="{{ repository.html_url }}">{{ repository.name }}</a></h2>
	<div>{{ repository.description }}</div>
  </div>
{% endfor %}
