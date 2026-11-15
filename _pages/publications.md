---
layout: page
permalink: /publications/
title: Publications and Global Research Impact
description: "Papers are arranged in a descending chronological order based on their publication year. You can also find my publications on Google Scholar. I am grateful that my work has reached researchers across 93 countries. This map visualizes the affiliations of scholars who have cited my publications (Last updated: Nov. 15, 2026)."
nav: true
nav_order: 1
---

<div style="width: 100%; margin: 2rem 0;">
  <p style="font-size: 0.9rem; color: grey;"></p>
  <iframe src="/NR/assets/html/citation_map.html" 
          style="width: 100%; height: 500px; border: 1px solid var(--global-divider-color); border-radius: 6px;" 
          loading="lazy">
  </iframe>
</div>

<!-- _pages/publications.md -->
<div class="publications">

{% bibliography -f {{ site.scholar.bibliography }} %}

</div>
