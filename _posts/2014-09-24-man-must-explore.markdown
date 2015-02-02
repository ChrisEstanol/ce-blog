---
layout:     post
title:      "Blogging with jekyll 101"
subtitle:   "Getting this adventure off the ground"
date:       2014-09-24 12:00:00
author:     "Christophe Estanol"
---

Let's get started this blogging thing with a bit of inside on how to get started with Jekyll and host it for free with github pages.
It took me a while to figure out the whole process and I actually had to use [Cloudflare](https://cloudflare.com/) to get over a 302 error.

## Jekyll

[Jekyll](http://Jekyllrb.com) is a static site generator widely used amongst developer, designer or anybody just wiling to write from a simple editor like Sublime or Atom. Jekyll uses [Markdown](http://daringfireball.net/projects/markdown/) syntax to format the post into HTML. It is pretty lightweight, easy to use but the best thing about Jekyll is that it can be hosted with [Github pages](https://pages.github.com/) for free.
Jekyll has a very comprehensive documentation to get you started in no time. To start exploring how Jekyll works
just run the following [commands](http://jekyllrb.com/docs/quickstart/).
The easiest way to start your blog is to fork an existing repo. I had a look around a made a list of the best templates available.

### Here are my top 10 jekyll themes/repos:

* [Jekyll incorporated](http://incorporated.sendtoinc.com/)
* [Hyde](http://andhyde.com/)
* [Poole](http://demo.getpoole.com/)
* [Pixyll](http://pixyll.com/)
* [Clean Blog](http://ironsummitmedia.github.io/startbootstrap-clean-blog-jekyll/)
* [Shiori](http://ellekasai.github.io/shiori/)
* [Jekyll clean](http://scotte.github.io/jekyll-clean/)
* [Cahoon](https://github.com/arnp/cahoon)
* [Jekyll now](http://www.jekyllnow.com/)
* [Jekyll Material Design](https://github.com/sentenza/jekyll-material-design)

This blog is based on [Clean Blog](http://ironsummitmedia.github.io/startbootstrap-clean-blog-jekyll/) with a [simple side-bar](http://ironsummitmedia.github.io/startbootstrap-simple-sidebar/) . I wanted a website based on [Bootstrap](http://getbootstrap.com) so I could keep on adding on top of a responsive framework I am  familiar with.

### Jekyll structure:

This is the basic Jekyll layout

```
.
├── _config.yml
├── _drafts
|   ├── begin-with-the-crazy-ideas.textile
|   └── on-simplicity-in-technology.markdown
├── _includes
|   ├── footer.html
|   └── header.html
├── _layouts
|   ├── default.html
|   └── post.html
├── _posts
|   ├── 2007-10-29-why-every-programmer-should-play-
|   └── 2009-04-26-barcamp-boston-4-roundup.textile
├── _data
|   └── members.yml
├── _site
└── index.html

```
The _config.yml file contains some general settings about  your page

```yaml
title: Christophe Estanol - Blog
email: hi@chrisestanol.com
description: "Write your site description here."
baseurl: ""
url: "http://chrisestanol.com"
twitter_username: chrisestanol
github_username:  chrisestanol
```
The folders **_includes** and **layouts** contain the building blocks of your blog. You can create multiples layouts using [Liquid](http://liquidmarkup.org/) templating engine. The **_drafts** and **_posts** folders hold your creative work to be released to the world.
When you run Jekyll the **_site** folder is created which contains all of your HTML nicely formated. You can add to this structure any folder you may need like css, js, img as well as additional static pages about.html, contact.html etc.

### Front Mater

```yaml
---
title: YAML Front Matter
description: A very simple way to add structured data to a page.
---
```
Jekyll uses [YAML Front Matter](http://jekyllrb.com/docs/frontmatter/) to add structured data like variables or any particular layout to a page. The variables can be created and then used within the page using [Liquid](http://liquidmarkup.org/).

```html
---
layout:     post
title:      "Dinosaurs are extinct today"
subtitle:   "because they lacked opposable thumbs and the brainpower to build a space program."
date:       2014-06-10 12:00:00
author:     "Dr. Jekyll"
---

<div class="post-heading">
  <h1>{{ page.title }}</h1>
  {% if page.subtitle %}
  <h2 class="subheading">{{ page.subtitle }}</h2>
  {% endif %}
  <span class="meta">{{ page.date | date: "%B %-d, %Y" }} </span>
</div>

```



