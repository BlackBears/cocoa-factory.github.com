---
layout: post
title: "We're moving (blogs)"
date: 2012-09-06 15:57
comments: false
categories: 
---

## Goodbye Wordpress, hello Octopress ##

We're in the process of moving all of our company pages and weblog to Octopress hosted through Github.  For several years, we maintained a presence on a dedicated site mostly setup with Wordpress.  Why the switch?

It came down to several factors:

* Lightweight vs. heavyweight solutions

	Wordpress is a great blogging platform; but there's a lot of "stuff" to manage.  Further, it's a dynamic content system - meaning, your content has to be generated from a database each time the page is rendered.  Octopress, instead, serves static pages.  There's something appealingly simple about static content.

* Workflow

	We develop software first, and blog second.  Writing about our work is important; but it has to fit into our workflow.  Since we are always working in `git` using Octopress feels very comfortable.

* Kick-start

	Working through the challenges of getting Octopress installed (and they were significant) was paradoxically a motivator to get down to writing more.  I mean - having spent all this time getting Octopress installed, I'm not about to let the effort be wasted!

* Comments

	I got tired of seeing queues of comments 99.9% of which was spam.  I'm sure Wordpress has better ways of dealing with comment spam - but I decided it was just easier to turn comments off on the new weblog.  The thing is, the real discussion isn't happening on the weblog; it's happening on Twitter and everywhere else.  You really should read Matt Gemmell's [comments on comments](http://mattgemmell.com/2011/11/29/comments-off/) about the phenomenon.

## Installing Octopress ##

Octopress installation was a little cumbersome owing mainly to difficulty with versioning of `rvm`.  After reading about the difficulties of installing on OS X 10.8, I decided to install it on Ubuntu 12.04 which I run as a virtual machine alongside Mountain Lion.  Again, it was mainly about making sure that the versioning of `rvm` was correct.  There are detailed instructions online, but I would add a couple of pointers:

We used [a tutorial](http://www.lennu.net/2012/05/11/octopress-installation-in-ubuntu-12-dot-04-with-rsync/) on lennu.net to get started.  Just make sure you get `rvm 1.9.2` installed.  If you install the latest stable version, you'll come to grief later in the installation.

Apart from that, all of the instructions are at Github and on the lennu.net tutorial.
