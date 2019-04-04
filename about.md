---
layout: page
title: About
permalink: /about/
---

I am a software developer and researcher from Pretoria, South Africa.

{%- assign social = site.minima.social_links -%}

<ul class="social-media-list">
  {%- if social.github -%}<li><a href="https://github.com/{{ social.github | cgi_escape | escape }}" title="{{ social.github | escape }}"><svg class="svg-icon grey"><use xlink:href="{{ '/assets/minima-social-icons.svg#github' | relative_url }}"></use></svg></a></li>{%- endif -%}
  {%- if social.linkedin -%}<li><a href="https://www.linkedin.com/in/{{ social.linkedin | cgi_escape | escape }}" title="{{ social.linkedin | escape }}"><svg class="svg-icon grey"><use xlink:href="{{ '/assets/minima-social-icons.svg#linkedin' | relative_url }}"></use></svg></a></li>{%- endif -%}
  {%- if social.twitter -%}<li><a href="https://twitter.com/{{ social.twitter | cgi_escape | escape }}" title="{{ social.twitter | escape }}"><svg class="svg-icon grey"><use xlink:href="{{ '/assets/minima-social-icons.svg#twitter' | relative_url }}"></use></svg></a></li>{%- endif -%}
</ul>
