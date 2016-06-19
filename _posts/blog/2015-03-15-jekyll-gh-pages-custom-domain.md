---
layout:     post
title:      "Jekyll - Hosting and domain"
subtitle:   "Host your site with github page and configure a custom domain name"
header-img: "img/post/octocat.jpg"
thumb-img:  "img/post/thumb/octocat-thumb.jpg"
date:       2015-03-15 12:00:00
author:     "Christophe Estanol"
share:      True
comments:   True
category:   blog
sitemap:
  lastmod: 2015-03-15
  priority: 0.7
  changefreq: 'monthly'
---

As Github users we are all entitled an unlimited number of repositories to hold on our precious code and make it easy to control versions of our builds.
Github also enable hosting of static websites which means that a basic bootstrap webpage can be hosted for free thanks to github. Jekyll, which is also technicaly a static website comes in handy and can also be hosted for free thanks to our friends at github.

Isn't it a relief to know that you won't have to search for a hosting provider that is both performant and cheap, set up an FTP to upload your website and do that everytime you make the slightest update...

This is why Jekyll has been so popular because if you are already familiar with git and github updating your portfolio or publishing a blog post your is just a commit away.

### Github Pages

There are two ways to host your webpage with [Github](https://pages.github.com/). There are User/Organisations pages or Project pages. Here are the main differences between the two:

* You can host your main website on ``http(s)://<username>.github.io``, where username is your username (or organisation name) on GitHub and deploy your projects on ``http(s)://<username>.github.io/<projectname>``.
* You only have one user page but you can host an unlimited amount of project pages
* For User pages you need to name you repository ``<username>.github.io`` for github to publish it as your web page. For project pages you can give it the name you like.
* User/Organisations page will remain on the master branch whereas Project pages will need to be pushed on a special branch named ``gh-pages``. You will then need to set up gh-pages as your default page.

<img src="{{ site.baseurl }}/img/gh-pages.png" alt="gh-pages branch">

You can get more info about the different hosting modes on [github help](https://help.github.com/articles/user-organization-and-project-pages/) page.

### Custom Domain

You can use custom domain names with both Project pages and User pages.
The set up is fairly easy:

1. Create a [CNAME](https://help.github.com/articles/adding-a-cname-file-to-your-repository/) document
2. Set up your [DNS](https://help.github.com/articles/tips-for-configuring-an-a-record-with-your-dns-provider/)

Here is an example of my DNS settings with Cloudflare
<img src="{{ site.baseurl }}/img/cloudflare.png" alt="gh-pages branch">

That's all you need!
