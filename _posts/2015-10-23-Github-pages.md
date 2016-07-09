---
layout: post
title: Managing repository websites without having to checkout the gh-pages branch (by SMM)
excerpt: "How you manage a project website in GitHub."
categories: articles
tags: git, website
---

* Table of Contents
{:toc}

## Managing repository pages in Github

Github enables you to have website for your repositories. 
Managing these websites takes a little bit of getting used to, because you will have to be familiar with banches in git. 
The post explains one way to manage a master branch for the actual software and a gh-pages branch for the website. 

### Setting up a website branch

In guthub, the repository website is always read from a branch called *gh-pages*. 
The first thing you need to do to set up a repository website is to create this branch. 

You can go here: https://pages.github.com/ to see the instructions. 

In this example, we are going to make a project site, and start from scratch. 

Making a new branch in the repository is the easy bit!

### Setting up your repositories

Once you have created a branch, you need to manage both the master and the gh-pages branches. 
You might not want them to sit in the same place.

We are going to follow the instructions here: https://gist.github.com/chrisjacob/833223

1. First, make sure your master branch is up to date on github and then delete it locally. You are going to clone it into a master subdirectory. 
I am going to do this with the LSDTT_book repository. 
2. On github, create a gh-pages branc. 
3. In the LSDTT_book repoository, make two directories: *master* and *gh-pages*.
4. Now clone the main repository into the *master* repo from the LSDTT_book directory
{% highlight console %}
$ pwd
home/Git_projects/LSDTT_book/
$ git clone https://github.com/LSDtopotools/LSDTT_book.git master
{% endhighlight %}
5. Now clone the repo *AGAIN*, but this time into the gh-pages directory
{% highlight console %}
$ pwd
home/Git_projects/LSDTT_book/
$ git clone https://github.com/LSDtopotools/LSDTT_book.git gh-pages
{% endhighlight %}
6. Check out the gh-pages branch
{% highlight console %}
$ cd gh-pages
$ pwd
home/Git_projects/LSDTT_book/gh-pages
$ git git checkout origin/gh-pages -b gh-pages
{% endhighlight %}
7. Now delete the master branch here:
{% highlight console %}
$ git branch -d master
*This step is mainly to ensure that you don't accidentally commit something to master while you are in the gh-pages directory*. 
It is not *needed* but is a useful precaution. 
{% endhighlight %}
8. Now you can push changes to your gh-pages branch from this repo without having to check it out each time
{% highlight console %}
$ git push -u origin gh-pages. 
{% endhighlight %}
