---
layout:     post
title:      "Rails - Tagging with autocomplete"
subtitle:   "How to integrate act_as_taggable gem with Chosen JS library for autocomplete"
header-img: "img/drjekyll.jpg"
thumb-img:  "img/drjekyll-thumb.jpg"
date:       2015-02-03 12:00:00
author:     "Christophe Estanol"
share:      True
comments:   True
category:   blog
---

For a project I was recently working on I had to set up a simple tagging system along with autocompletion. Amongst many options I opted for a great gem called act_as_taggable and the [chosen](https://github.com/harvesthq/chosen) JQuery library.

We will be creating a sample application that manages products, this could be the base for a store for example. You can find this [example](https://github.com/ChrisEstanol/tagging_autocomplete) in my github.

Let's start by creating our new application

```bash
rails new tagging_autocomplete -T
```
**-T** will omit all the test framework.

Then let's add our gems and run bundle install

```ruby
gem 'bootstrap-sass'
gem 'acts-as-taggable-on'
gem 'rails-assets-chosen'
```

One way to add external JQuery plugins with your assets is to use [Rails Assets](https://rails-assets.org/). *It automatically converts the packaged components into gems that are easily droppable into your asset pipeline and stay up to date*. I came across Rails assets in this [Go Rails](https://gorails.com/episodes/rails-assets) episode an I would recommend to use it every time you need to include external JS libraries into your rails application.

Next we'll need to create our CRUD to manage products and taggs

```bash
rails g scaffold Product name:string description:text
rails g scaffold Tag name:string
```

To create a leaner Scaffold without helpers, test, assets you can:
- add `--no-helper --no-assets --no-controller-specs --no-view-specs --no-test-framework` after your scaffold command or
- include in **config/application.rb**
```ruby
config.generators do |g|
  g.assets            false
  g.helper            false
  g.test_framework    nil
  g.jbuilder          false
end
```
