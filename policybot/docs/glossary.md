---
layout: default
title: Glossary
nav_order: 10
description: "Definitions of all terms and acronyms used in the Policy Bot design guide."
permalink: /docs/glossary/
---

# Glossary
{: .no_toc }

All terms and acronyms used in this design guide, listed alphabetically.
{: .fs-6 .fw-300 }

---

<div class="glossary-search-bar" style="margin-bottom: 1.5rem;">
  <input 
    type="text" 
    id="glossary-filter" 
    placeholder="🔍 Filter terms..." 
    style="width:100%;padding:0.6rem 1rem;font-size:1rem;border:1px solid #e1e4e8;border-radius:6px;outline:none;"
    oninput="filterGlossary(this.value)"
  />
</div>

<dl id="glossary-list">
{% assign sorted_terms = site.data.glossary | sort: "term" %}
{% for entry in sorted_terms %}
<div class="glossary-entry" data-term="{{ entry.term | downcase }}">
  <dt id="{{ entry.term | slugify }}" style="font-weight:700;font-size:1.05rem;margin-top:1.25rem;color:#0969da;">
    {{ entry.term }}
    <a href="#{{ entry.term | slugify }}" style="margin-left:0.4rem;color:#6e7781;font-size:0.85rem;text-decoration:none;" aria-label="Link to {{ entry.term }}">#</a>
  </dt>
  <dd style="margin-left:1.5rem;margin-top:0.25rem;color:#24292f;">{{ entry.definition }}</dd>
</div>
{% endfor %}
</dl>

<p id="no-results" style="display:none;color:#6e7781;font-style:italic;">
  No matching terms found.
</p>

<script>
function filterGlossary(value) {
  const query = value.toLowerCase().trim();
  const entries = document.querySelectorAll('.glossary-entry');
  let visible = 0;
  entries.forEach(function(entry) {
    const term = entry.getAttribute('data-term') || '';
    const definition = entry.querySelector('dd') ? entry.querySelector('dd').textContent.toLowerCase() : '';
    if (!query || term.includes(query) || definition.includes(query)) {
      entry.style.display = '';
      visible++;
    } else {
      entry.style.display = 'none';
    }
  });
  document.getElementById('no-results').style.display = visible === 0 ? '' : 'none';
}
</script>

---

## Quick Reference Index

Jump to terms starting with:

{% assign letters = "A,B,C,D,E,F,G,H,I,J,K,L,M,N,O,P,Q,R,S,T,U,V,W,X,Y,Z" | split: "," %}
<div style="display:flex;flex-wrap:wrap;gap:0.5rem;margin:1rem 0;">
{% for letter in letters %}
  {% assign has_term = false %}
  {% for entry in site.data.glossary %}
    {% if entry.term | slice: 0 | upcase == letter %}
      {% assign has_term = true %}
    {% endif %}
  {% endfor %}
  {% if has_term %}
  <a href="#{{ sorted_terms | where_exp: "e", "e.term | slice: 0 | upcase == letter" | first | map: "term" | first | slugify }}"
     style="padding:0.25rem 0.6rem;background:#f6f8fa;border:1px solid #d0d7de;border-radius:4px;font-family:monospace;font-size:0.95rem;">
    {{ letter }}
  </a>
  {% else %}
  <span style="padding:0.25rem 0.6rem;color:#d0d7de;font-family:monospace;font-size:0.95rem;">{{ letter }}</span>
  {% endif %}
{% endfor %}
</div>
