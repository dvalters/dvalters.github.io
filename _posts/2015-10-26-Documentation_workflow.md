---
layout: post
title: Documentation an gh-pages workflow (by SMM)
excerpt: "Workflow for managing documentation pages."
categories: articles
tags: git, website, gh-pages, documentation
---

* Table of Contents
{:toc}

# Workflow for documentation pages

After quite a bit of trial and error, I have settled on a documentation workflow I'm happy with. 

## Setting up the repositories. 

To start, you need to set up a documentation structure that forms a static html site. 
This could be a variety of things, such as asciidocotr, dOxygen, Sphynx, jekyll, etc. 

Then make a directory structure so there is a folder for both the master and gh-pages branches of the code. 

I found having seperate folders minimizes the chances of having merge conflicts. 

So for example, you might have a repository that is called *My_docs* and in it are two subfolders *master* and *gh-pages*

For example:

{% highlight console %}
$ pwd
home/Git_projects/My_docs/
$ ls
master gh-pages
{% endhighlight %}

## Keep the source files in the master branch

In the master branch, keep the source files for the static website. For example, I keep all my asciidoctor source files in the *master* branch. 

## Build the website, and keep the website files in the *gh-pages* branch. 

I use asciidoctor to build webistes, but the built website is not tracked in the *master* folder. 
Instead, when I build the website, I then copy the html and image files over to gh-pages and commit them *ONLY* to the gh-pages branch. 

## Making sure you don't have conflicts. 

To avoid conflicts, it is helpful to only have the gh-pages branch in the gh-pages folder, and I also find it useful to only have
files needed for building the website in the gh-pages branch. 

For the *master* branch, you just work as normal. 

For the gh-pages branch you
1. Clone the master into the gh-pages branch
2. Check out the gh-pages branch
{% highlight console %}
$ pwd
home/Git_projects/My_docs/gh-pages
$ git git checkout origin/gh-pages -b gh-pages
{% endhighlight %}
3. Now delete the master branch in the gh-pages folder:
{% highlight console %}
$ git branch -d master
{% endhighlight %}
*This step is mainly to ensure that you don't accidentally commit something to master while you are in the gh-pages directory*. 
It is not *needed* but is a useful precaution. 
4. Copy across html and index files every time you build the website. 