---
title: "Multilingual website with Jekyll on GitHub Pages"
categories: cms
tags: [ jekyll ]
permalink: /jekyll-multilingual-website/
---
As soon as you have a Jekyll website on GitHub pages, the question arises how
to make it multilingual. Of course, there is Google Translate but is there a
better way?

<!--more-->

# A website using a multilingual plugin

There are some multilingual plugins for Jekyll like
[jekyll-multiple-languages-plugin](https://github.com/Anthony-Gaudino/jekyll-multiple-languages-plugin) or
[polyglot](https://polyglot.untra.io/).

However, the [list of Jekyll plugins supported by
GitHub](https://pages.github.com/versions/) shows that those plugins are
not supported.

# A website without a multilingual plugin

So we need an approach without plugins and after some Google research I found these articles:

* [Making Jekyll multilingual](https://www.sylvaindurand.org/making-jekyll-multilingual/)
* [Build a multilingual website with jekyll](http://chocanto.me/2016/04/16/jekyll-multilingual.html)
* [Creating a Multilingual Blog With Jekyll](https://forestry.io/blog/creating-a-multilingual-blog-with-jekyll/)

In all these articles the idea is to:
* store all pages in different languages in the same website (repository)
* create different language folders
* add administration to the pages or configuration to make it work (for example the language of a page)

But these are the disadvantages:
* the paginate plugin does not longer work
* you can not display simply all pages or posts but you must filter on them having the same page language 
* the [Minimal Mistakes theme](https://mmistakes.github.io/minimal-mistakes) is able to do a translation but only for the language system setting (site.locale).

So, if I wanted to use this method it would mean I had to override a lot of
files from the Minimal Mistakes theme, brrr.

# Every website having its own language

As it often happens, a night's rest came to my rescue. What if I would create
a website for each language I needed, so one in Dutch, one in English,
etcetera?

The advantages compared with the previous method:
* the paginate plugin will still work
* no language related changes to the Minimal Mistakes theme files necessary

The disadvantages:
* some duplication of configuration files (i.e. _config.yml, Gemfile)
* the necessity to work with a baseurl (site.baseurl)
* the creation of extra repositories
* some attention needed with site URL's

Please note that the content files (posts or pages) have to be translated in
either method, so I do not count that as duplication.

In my opinion the benefits outweigh the disadvantages.

# Repositories used

The next question was which Github repositories to use? There is already a
User (or Organization) site so you could make that the main website and the
translated sites would be Project sites. But I decided to have my User site
just display the most relevant projects including:
* My English business Blog (named blog)
* My Dutch personal Blog (named blog-nl)

# How site URL's are constructed

For a better understanding I print the website tree (using tree -A _site/) first:

```
_site/  
├── 404.html  
├── Rakefile  
├── about  
│   └── index.html  
├── assets  
│   ├── css  
│   │   └── main.css  
│   ├── images  
│   │   └── bio-photo.jpg  
│   └── js  
│       ├── _main.js  
│       ├── lunr  
│       │   ├── lunr-en.js  
│       │   ├── lunr-gr.js  
│       │   ├── lunr-store.js  
│       │   ├── lunr.js  
│       │   └── lunr.min.js  
│       ├── main.min.js  
│       ├── plugins  
│       │   ├── gumshoe.js  
│       │   ├── jquery.ba-throttle-debounce.js  
│       │   ├── jquery.fitvids.js  
│       │   ├── jquery.greedy-navigation.js  
│       │   ├── jquery.magnific-popup.js  
│       │   └── smooth-scroll.js  
│       └── vendor  
│           └── jquery  
│               └── jquery-3.4.1.js  
├── categories  
│   └── index.html  
├── contact  
│   └── index.html  
├── feed.xml  
├── getting-started  
│   └── index.html  
├── index.html  
├── jekyll-multilingual-website  
│   └── index.html  
├── jython-red  
│   └── index.html  
├── oradumper  
│   └── index.html  
├── posts  
│   └── index.html  
├── robots.txt  
├── search  
├── sitemap  
│   └── index.html  
├── sitemap.xml  
└── tags  
    └── index.html
```

In [blog/_config.yml](https://github.com/gpaulissen/blog/blob/master/_config.yml) these are the relevant URL settings:

url                  : 'https://gpaulissen.github.io'  
baseurl              : '/blog'

This baseurl is prepended by Jekyll to (most of) the relative URL's (an URL without
://) used in your posts or pages. There are some exceptions:
* the Markdown link or image url in a page
* the Minimal Mistakes theme url settings in _config.yml like author url's

For these exceptions there are two cases:
1. when a relative url starts with a / (e.g. /assets/images/bio-photo.jpg)
2. when a relative url does **not** start with a / (e.g. ../assets/images/bio-photo.jpg)

The first case means that the absolute url will be the site.url plus the relative url, thus:
1. https://gpaulissen.github.io/assets/images/bio-photo.jpg
2. https://gpaulissen.github.io/../assets/images/bio-photo.jpg

So when the current page is
https://gpaulissen.github.io/blog/getting-started/index.html, you can see that
the first relative URL is **wrong** and the second is **okay**.

# Conclusion

Using separate websites (GitHub repositories) is a simple way to get multilingual websites.

Actually, for me the English Blog site was business orientated and the Dutch
Blog site was personal so I should already have them separated at the start, 😞.

But then I would not have created this Blog, 😊.

