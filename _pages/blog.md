---
title: "전체 글 목록"
layout: archive
permalink: /blog/
author_profile: true
---

{% assign posts = site.posts %}
{% assign entries_layout = page.entries_layout | default: 'list' %}
<div class="entries-{{ entries_layout }}">
  {% include documents-collection.html entries=posts type=entries_layout %}
</div>