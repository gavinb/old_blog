---
layout: page
title: Gavin Baker
---
{% include JB/setup %}

This is the blog of Gavin Baker.  Here I publish articles, projects and musings on Software and the industry.

## Blog

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## Articles

Featured articles published on this site include:

### Boost

 * Threading
 * Mutexes
 * Condition variables

Other articles and opinion pieces are in the blog.

## Projects

Here I publish a few little projects I work on from time to time.

## Contact

If you found any of the above interesting, you should follow me on Twitter @gavinb !
