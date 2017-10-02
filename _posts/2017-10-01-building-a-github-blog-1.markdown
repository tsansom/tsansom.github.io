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
$ gem install jekyll bundler
```

*Note: This tutorial assumes that you have __git__ set up on your system. If you
  do not already have it see [Setting Up
  Git](https://help.github.com/articles/set-up-git/).*

## Setting up the Blog
#### Local Setup
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
$ bundle exec jekyll serve
```
Then, in a browser, navigate to `localhost:4000` and you will see something like
  this:

![jekyll-template]({{ site.url }}/images/jekyll_template.png)

Changes made to any of the files in the project folder will be automatically
  updated (**except for _config.yml**) so you can just refresh the page in the
  browser and your changes will be there.

#### Personalizing the Basics
To add your personal information, blog title, url, etc., open the file
  **_config.yml** and replace the existing information there. You will have to
  restart the server after making changes to _config.yml.

#### Getting the Blog on Github
To make this blog live, we will first need to create the repository on
  [Github](https://github.com). Click `Create Repository` and name the repo
  `<username>.github.com`. You can add a description if you want but don't
  initialize with a `README.md` just yet. After you click `Create Repository`,
  follow the instructions to create a new repository on the command line:
```bash
$echo "# <username>.github.io" >> README.md
#this creates the readme file and adds the project name as the only line
$ git init
$ git add README.md
#this will add all the files to the git repo
$ git commit -m "first commit"
$ git remote add origin https://github.com/<username>/<username>.github.io.git
$ git push -u origin master
```
If that worked, add the rest of the files to the github repo:
```bash
$ git add .
$ git commit -m "adding blog files"
$ git push
```

After a few seconds, the site should be live at `https://<username>.github.io`.
  You will see the same Jekyll template as we saw earlier (unless you changed
  some fields in _config.yml).

Congratulations, you now have a fully operational blog. To make your first post,
  simply edit the lone markdown file in the folder _posts or add another
  markdown file with the title `YYYY-MM-DD-name-of-post.markdown`. You will
  need to add the yaml front matter at the top:
```markdown
---
layout: post
title:  "First Post"
date:   2017-10-01 05:49:27 -0500
categories: github blog jekyll
---
```
Then, add some markdown to the file, save it, serve the site with `bundle exec
  jekyll serve`, and navigate to `localhost:4000`. If you are satisfied with the
  changed, push the changes to your github repo (add, commit, push).

## Changing Themes
So far, so good. We've created a blog template which we can develop locally and
  deploy on github with just a few lines of code. Now, let's change the theme so
  our blog stands out from every other blog made with the default template.

This process is a bit tricky in the way that bundler and jekyll store and use
  the theme files. For instance, you will notice that there are currently no files
  in our blog repo that deal with how each one of our pages is formatted (*These
  files would normally live in the `_layouts` folder*). Also there's no css
  stylesheet. This is because some of the site's files are stored in the theme's
  gem and are hidden from our view. To override the defaults for any theme, we
  just need to have the similarly named file(s) that we want to override in our
  local repository. **This is because Jekyll looks for the necessary files in your
  site's folder first, and then looks for the files in the theme's gem.** We'll
  see this in action shortly.

When we created the jekyll template, the minima theme was used by default. To
  see where the rest of the files associated with the minima theme are stored,
  use:
```bash
$ bundle show minima
#show the path to minima theme
```
Some other helpful commands that might be useful for this process:
```bash
$ ls $(bundle show minima)
#list the contents of the minima theme folder
$ cd $(bundle show minima)
#change directory to the minima folder
$ open $(bundle show minima)
#open the minima theme folder in a Finder window
```
I'm going to be using [`jekyll-theme-slate`](https://github.com/pages-themes/slate)
  for my blog, but the process described here should work for other themes as
  well. First we need to edit the Gemfile to make sure that we have the theme's
  gem installed.

Edit **Gemfile** in your favorite editor:
1. comment out the line `gem "jekyll", "~>3.6.0"`
    - *version numbers will likely differ*
2. comment out the line `gem "minima", "~> 2.0"`
3. add this line `gem "jekyll-theme-slate"`
4. uncomment the line `# gem "github-pages", group: :jekyll_plugins`
5. comment out the last line if not on windows
    - It's just there becuase windows doesn't handle time zone info very well

Now let's install the theme's gem. Run the following:
```bash
$ bundle update
$ bundle install
```

Next we need to tell Jekyll which theme to use. Do this by editing the `_config.yml`
  file and changing the theme from `minima` to `jekyll-theme-slate`. Here's where
  I got confused the first time through. If you serve the site now with `bundle
  exec jekyll serve` you will get three build warnings saying that layouts for
  post, page, and home do not exist. The reason for this, as mentioned earlier,
  is that Jekyll is looking locally first and then in the theme's gem for files
  it needs. **But `jekyll-theme-slate` has no layouts for post, page, and home!**

If you look at the top of the file `about.md` you will see the yaml front matter
  that looks like this:
```markdown
---
layout: page
title: About
permalink: /about/
---
```
The line `layout: page` is where the layout for this particular page is set.
  Before the theme was changed, it found this file in `minima`'s gem. Now, since
  it's looking in `jekyll-theme-slate`'s gem, it's not there.

So what we need to do is copy the page, post, and home layout files from the
  minima theme's gem to our local repository. First, make a folder called `_layouts`
  in the top directory of your site:
```bash
$ mkdir _layouts
```
Next, copy the contents of minima's `_layouts` folder to your newly created folder:
```bash
$ cp $(bundle show minima)/_layouts/* _layouts/
```
Finally since we want to use the default template from the `jekyll-theme-slate`
  gem, remove the `defualt.html` file:
```bash
$ rm _layouts/default.html
```

<br>

Now if you serve the site with `bundle exec jekyll serve` you will see a very
  good looking blog homepage like this:

![slate_theme]({{ site.url }}/images/slate_theme.png)

## Enabling Comments
