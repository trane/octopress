---
layout: post
style: text
title: First post
---

#Hello.

My name is Andrew and this is my blog.
I suppose I should start off with something useful other than a *Hello World*
post, so I'll talk about the blogging software that I'm using:
[Jekyll](https://github.com/mojombo/jekyll), specifically a modified clone of
[\_layout](https://github.com/studiomohawk/jekyll-theme-the_minimum). I'm sure
the modifications will only grow more as time goes on.
Let's play with some markdown syntax...

I chose Jekyll because
* I enjoy the idea of statically serving my site
* I can setup [git post-hooks](http://blog.jpoz.net/2009/03/05/jekyll_blog_hooks.html) to *automagically* regenerate the site
* of my affinity towards [Ruby](http://www.ruby-lang.org/en/)

Let's try out some [pygments](http://pygments.org/) syntax highlighting. Here is
the command I used to delete the example posts:
{% highlight bash %}
find . -type f -name "*example*" -exec rm {} \;
{% endhighlight %}
I suppose we should talk about what this does, since *find* seems to be the most
under used (by newer users), yet powerful \*nix utilities *ever*.
{% highlight bash %}
find .            \ # search this directory and subdirectories
-type f           \ # search for only files (don't include directories)
-name "*example*" \ # only get files with the word example somewhere in them
-exec rm {}       \ # remove each file returned
\;                  # syntax to tell exec that we are done
{% endhighlight %}

##P.S.

Hopefully by the time you read this post the code will be in monospace, can't
seem to figure out why my changes to the css haven't shown up.

##P.P.S.
So, monospace is fixed and a bug with the code alignment on widescreens is fixed
as well. I'm glad that I let
[\_layout](https://github.com/studiomohawk/jekyll-theme-the_minimum) do the
heavy lifting.
