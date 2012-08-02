---
layout: post
author: dagingaa
title: How Comoyo built its blog with Jekyll and GitHub Pages
category: jekyll
---
_This post was originally posted on [Comoyo Engineering](http://comoyo.github.com/blog/2012/06/11/how-comoyo-built-its-blog/)._

![Comoyo Jekyll Blog Flow Chart](/assets/img/posts/comoyo_jekyll_blog.png)

When we set out to relaunch our company techblog, we had a few specific requirements:

1. The blog should require no form of monitoring or on-call time.
2. The blog must scale to meet (almost) any amount of traffic.
3. Writing a post, submitting it for review, and getting it published should take as little time as possible away from the developer.
4. All developers must be able to post, whenever they want to post.
5. Developers must be familiar with the tools used to create the blog.

Of course, we _could_ set up a dedicated Wordpress installation on one or several of our AWS-instances, and make it scalable with heavy caching and Varnish. However, this solution is prone to breakage, as we have seen with other Wordpress installations. Developers also have a tendency to not like WYSIWYG-editors, arguing that they create ugly and unreadable HTML. Not to mention that we would have to create users for every developer before they could start to blog, taking more time away from actual programming.

##Enter Jekyll##
Then we discovered Jekyll. [Jekyll is a simple, blog aware, static site generator](https://github.com/mojombo/jekyll). It is the main component behind [GitHub Pages](http://pages.github.com/), a free hosting solution from GitHub. GitHub takes your Jekyll application (and static files too!), runs it through its compiler, and hosts the results. They even have a [handy guide](https://help.github.com/articles/using-jekyll-with-pages) on how to create and publish your Jekyll application. Developers can fork, make pull requests, comment inline, use whatever editor they wanted, see diffs of content, and best of all, it needed little to no monitoring from our on-call engineer.

So with this in mind, I was tasked with creating the company blog using Jekyll, including a complete bootstrap stylesheet for Comoyo. I used [Initializr](http://www.initializr.com/) to create the basic structure, using HTML5 Boilerplate, Bootstrap 2.0.4, Modernizr, jQuery and LESS. Initial development was slow at first, mainly because I didn't know about the excellent bootstrap solution provided by [Jekyll Bootstrap](http://jekyllbootstrap.com/). After the [initial folder structure](https://github.com/mojombo/jekyll/wiki/usage) is set up, the rest was pretty straight forward. Our folder structure now looked something like this:

	.
	|-- _config.yml
	|-- _includes
	|   |-- article_header.html
	|   |-- author.html
	|   |-- authors.html
	|   `-- comments.html
	|-- _layouts
	|   |-- default.html
	|   `-- post.html
	|-- _posts
	|   |-- 2012-06-12-how-comoyo-built-its-blog.md
	|   `-- _1970-01-01-example.md
	|-- _site
	|-- assets
	|   |-- css
	|   |-- fonts
	|   |-- img
	|   |-- js
	|   `-- less
	|-- .gitignore
	|-- 404.html
	|-- favicon.ico
	|-- Gemfile
	|-- humans.txt
	|-- README.md
	`-- index.html

##Outsource everything##
With GitHub Pages, we no longer have a database, which would make it hard for ourselves to store comments, likes etc. However, as almost everyone else these days, we decided on using [Disqus](http://disqus.com/). The main advantage of outsourcing everything like this is that we don't have to spend much money ourselves keeping the blog up, making it easier to justify spending developers time on writing posts, not spending time trying to keep everything backed up, monitored and live. 

Of course, outsourcing has its drawbacks. Not controlling our own platform means that we risk unplanned downtime, data loss, sort of like [setting sail in a ship who's direction you no longer have control over](http://www.webdistortion.com/2012/06/11/dont-build-your-house-on-someone-elses-platform/). But since we use git, we will have multiple repos, thanks to the distributed nature of git. The risk of data loss is minimal. The only problem might be service outages at GitHub or Disqus, but then we would have bigger problems to deal with anyway.

##Making Jekyll overcome its hurdles##
As with any framework, Jekyll is not without its hurdles. The biggest issue we had was that [GitHub pages does not support Jekyll plugins](https://github.com/mojombo/jekyll/issues/325). Since we wanted to customize our blog with multiple authors, better frontpage view and more, this was now out of the question. Initially, we had used the excellent plugin [only_first_paragraph](https://github.com/sebcioz/jekyll-only_first_p) to create a nice concatenated index of all posts. The solution we chose was to simply forget about it for now, and later add some javascript to do the same. 

Another issue was that Jekyll is by default a single-author blogging platform. We needed support for multiple authors, preferably with author profiles and metadata. After some quick googling, [we found a solution](http://www.lostdecadegames.com/blog-author-attribution-using-jekyll/). We also created two simple includes, authors.html which lists a short bio for each author, and author.html that shows the author profile when reading a post.

{% highlight yaml %}
# _config.yml
authors:
  dagingaa:
    name: Dag-Inge Aas
    position: Software Developer Intern
    gravatar: d4acca0bcb24dc67644055d4e44a6b29
    web: http://www.daginge.com
    github: dagingaa
    twitter: daginge
    about: |    
      Summer intern at Comoyo. Likes front-end development, new technology
      and cake. Currently active as Head of IT for [ISFiT](http://www.isfit.org).
{% endhighlight %}

We could then get the authors array through site.authors and iterate through this as one would expect. 

##Getting Jekyll ready for production##
By now, we have about 15 less-files getting requested, parsed and compiled to css by less.js. For a production site, this is unacceptable. Jekyll does not have a built in css or js-compressor, so we have to do this ourself. I landed on using recess, the same tool that Twitter Bootstrap uses to compile and compress the less-files into a single css-style. For someone that has checked the source, you can see that we still load several .js and .css-files, but the situation is far better than what it was. In time, we will create a build script that can be run before pushing changes to GitHub that automatically combines all javascript and css.

In short, we wanted our blog to scale to demand, use the latest web technology, be insanely fast, and become a digital playground were developers can create, write and modify everything the way they prefer. We believe that this is the best way to motivate developers to write for a company techblog.

**Hi, we are Comoyo.**
