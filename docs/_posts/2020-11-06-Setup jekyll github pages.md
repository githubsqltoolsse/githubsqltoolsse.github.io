---
layout: posts
title:  "Jekyll setup!"
date:   2020-11-06 13:05:00 +0100
categories: jekyll-update
tags: first fourth
---

# Installations

# Add minimal misstakes theme

## Install the theme
In gem Gemfile add 
{% highlight ruby %}
gem "minimal-mistakes-jekyll"
{% endhighlight %}

In config.yml add

{% highlight yaml %}
remote_theme: "mmistakes/minimal-mistakes@4.21.0"

plugins:
  - jekyll-feed
  - jekyll-include-cache
  - jekyll-remote-theme
{% endhighlight %}


## Add main menu

Add a _data/navigation.yml file. In there add code with your menu settings.


{% highlight yaml %}

main:
  - title: "Quick-Start Guide"
    url: /docs/quick-start-guide/
  - title: "Posts"
    url: /year-archive/
  - title: "Categories"
    url: /categories/
  - title: "Tags"
    url: /tags/
  - title: "Pages"
    url: /page-archive/

{% endhighlight %}

## Syntax highligting
See which languages and tags to use on this page: [List of supported pages](https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers)

~~~
{% highlight sql %}
SELECT
  *
FROM
  MyTable
{% endhighlight %}
~~~

# Publish new code
{% highlight console %}
c:\...rebository\docs> bundle
{% endhighlight %}

{% highlight console %}
c:\...rebository\docs> bundle update
{% endhighlight %}

## Update program or gems

## Test run locally

{% highlight console %}
c:\...rebository\docs> bundle exec jekyll serve
{% endhighlight %}


## Publish online

# Notes

I got build warnings saying that the layout couldn't be found. Checkout on the themes github page in layouts folder that it actually exist an layout named what you are using in your page or post. Apparantly it has been a change in jekyll from post to posts and page to pages.

