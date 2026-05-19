---
permalink: /search/
title: "Search"
excerpt: "Search across all pages and content."
layout: single
author_profile: false
---

<div class="search-bar">
  <input type="text" id="search-box" class="search-box" placeholder="Type to search papers, authors, events..." autofocus />
  <button id="search-go" class="search-go"><i class="fas fa-search"></i> Search</button>
</div>
<div id="search-results" class="search-results"></div>
<div id="search-empty" class="search-empty">Enter a keyword to search across the LLM4EC website.</div>

<script>
(function() {
  var pages = [
    {% for page in site.pages %}
      {% unless page.url contains '.xml' or page.url contains '.css' or page.url contains '.js' or page.url contains 'feed' %}
        {% assign content = page.content | strip_html | strip_newlines | escape_once | jsonify %}
        {% assign title = page.title | default: "LLM4EC" | escape_once | jsonify %}
        {% assign excerpt = page.excerpt | default: "" | strip_html | strip_newlines | escape_once | jsonify %}
        {
          title: {{ title }},
          url: "{{ site.baseurl }}{{ page.url }}",
          content: {{ content | default: "''" }},
          excerpt: {{ excerpt | default: "''" }}
        },
      {% endunless %}
    {% endfor %}
  ];

  var box = document.getElementById('search-box');
  var goBtn = document.getElementById('search-go');
  var results = document.getElementById('search-results');
  var empty = document.getElementById('search-empty');

  function doSearch() {
    var query = box.value.trim().toLowerCase();
    if (query.length < 2) {
      results.innerHTML = '';
      empty.style.display = 'block';
      return;
    }

    var hits = [];
    pages.forEach(function(p) {
      if (!p.content && !p.title) return;
      var text = (p.title + ' ' + p.content).toLowerCase();
      if (text.indexOf(query) === -1) return;

      var score = 0;
      var titleLower = p.title.toLowerCase();
      if (titleLower.indexOf(query) !== -1) score += 100;
      var idx = -1, count = 0;
      while ((idx = text.indexOf(query, idx + 1)) !== -1) count++;
      score += Math.min(count, 20);

      var snippet = '';
      var pos = text.indexOf(query);
      if (pos > -1) {
        var start = Math.max(0, pos - 80);
        var end = Math.min(text.length, pos + query.length + 120);
        snippet = (start > 0 ? '...' : '') + text.substring(start, end) + (end < text.length ? '...' : '');
      }

      hits.push({ page: p, score: score, snippet: snippet });
    });

    hits.sort(function(a, b) { return b.score - a.score; });

    if (hits.length === 0) {
      results.innerHTML = '<p class="search-no-results">No results found for "' + query + '".</p>';
      empty.style.display = 'none';
    } else {
      var html = '<p class="search-count">Found ' + hits.length + ' result(s):</p>';
      hits.forEach(function(h) {
        html += '<div class="search-hit">' +
          '<a class="search-hit__title" href="' + h.page.url + '">' + h.page.title + '</a>' +
          '<div class="search-hit__snippet">' + h.snippet + '</div>' +
          '</div>';
      });
      results.innerHTML = html;
      empty.style.display = 'none';
    }
  }

  box.addEventListener('input', doSearch);
  goBtn.addEventListener('click', doSearch);
  box.addEventListener('keydown', function(e) {
    if (e.key === 'Enter') { e.preventDefault(); doSearch(); }
  });
})();
</script>

<style>
.search-bar {
  display: flex;
  gap: 0.5em;
}
.search-box {
  flex: 1;
  padding: 0.9em 1.2em;
  font-size: 1.1em;
  border: 2px solid #ddd;
  border-radius: 8px;
  outline: none;
  transition: border-color 0.2s;
  box-sizing: border-box;
}
.search-box:focus {
  border-color: #1a4b77;
}
.search-go {
  padding: 0.9em 1.5em;
  font-size: 1.05em;
  font-weight: 600;
  background: #1a4b77;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  white-space: nowrap;
  transition: background 0.2s;
}
.search-go:hover {
  background: #1a8a8a;
}
.search-empty {
  color: #999;
  text-align: center;
  margin-top: 2em;
}
.search-count {
  color: #666;
  font-size: 0.9em;
  margin: 1em 0 0.5em;
}
.search-no-results {
  color: #999;
  text-align: center;
  margin-top: 2em;
}
.search-hit {
  margin: 1.2em 0;
  padding-bottom: 1em;
  border-bottom: 1px solid #eee;
}
.search-hit__title {
  font-size: 1.1em;
  font-weight: 600;
  color: #1a4b77;
  text-decoration: none;
}
.search-hit__title:hover {
  text-decoration: underline;
}
.search-hit__snippet {
  margin-top: 0.3em;
  font-size: 0.9em;
  color: #555;
  line-height: 1.5;
}
</style>
