---
layout: post
comments: true
title:  "Building a Github Blog with Jekyll and Ruby"
date:   2017-10-01 05:49:27 -0500
categories: github blog jekyll theme comments
---

In this three-part tutorial I'll walk you through how to build a Github blog
  from scratch with Jekyll and Ruby. While most of this is pretty straight
  forward, there were a few things I encountered that were a bit tricky to
  understand and/or implement.

*Note: I'm setting this up on OS X so a few of the commands are system specific.*

[Part 0 - Prerequisites](#prerequisites)  
[Part 1 - Setting up the Blog](#setting-up-the-blog)  
[Part 2 - Changing Themes](#changing-themes)  
[Part 3 - Enabling Comments](#enabling-comments)  

## Prerequisites
#### Install RVM and Ruby
**RVM: Ruby Version manager** lets you switch between **Ruby** versions without
  changing the system version. Install them both with:
```bash
$ \curl -sSL https://get.rvm.io | bash -s stable --ruby
```
Before we can use RVM we ned to load its funtions with:
```bash
$ source /Users/<username>/.rvm/scripts/rvm
```

#### Install Jekyll and Bundler Gems
**Jekyll** is the backbone of this tutorial. It's a blog-aware, static site
  generator that handles the painful task of editing HTML and CSS files. We
  Can just give it a markdown file and it will give us a fully formatted blog
  post. **Bundler** provides version consistency for Jekyll projects by tracking
  and installing the exact gems that are needed for you system and setup.

They can be installed simultaneously with:
```bash
$ gem install jekyll Bundler
```

*Note: This tutorial assumes that you have __git__ set up on your system. If you
  do not already have it see [Setting Up
  Git](https://help.github.com/articles/set-up-git/).*

## Setting up the Blog
Personally, I prefer to have a local development environment to test everything
  and make sure the formatting and functionality acts as expected before making
  it live. Luckily, Jekyll makes this extremely easy.

Since we are building a blog on Github, we'll need to use the default Github-pages
  domain name format, `<username>.github.io`. To start a new Jekyll project, simply
  type:
```bash
$ jekyll new <username>.github.io
$ cd <username>.github.io
```
If everything went well, we can see the template site locally by typing:
```bash
$ jekyll serve
```
Then, in a browser, navigate to `localhost:4000` and you will see something like
  this:

![jekyll-template]({{ site.url }}/images/jekyll_template.png)

## Changing Themes

## Enabling Comments
