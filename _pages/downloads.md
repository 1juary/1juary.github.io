---
permalink: /downloads/
title: "Downloads"
excerpt: "Downloadable resources from Juary Wang"
author_profile: true
---

<span class='anchor' id='downloads'></span>

# 📥 Downloads

{% assign files = site.data.downloads %}

{% if files and files.size > 0 %}
  <div class="downloads-list">
  {% for item in files %}
    <div class="downloads-list__item">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
      <a href="{{ item.file }}" class="downloads-list__link">
        <i class="fas fa-download" aria-hidden="true"></i> Download
      </a>
    </div>
  {% endfor %}
  </div>
{% else %}
  <p style="text-align: center; padding: 3em 0; color: #7a8288;">
    No files available yet. Check back soon!
  </p>
{% endif %}
