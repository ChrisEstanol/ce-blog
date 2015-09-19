---
layout:     post
title:      "Spree - Custom front end "
subtitle:   "How to customize Spree front end and create a new landing page and responsive header"
header-img: "img/spree-custom.jpg"
thumb-img:  "img/spree-custom-thumb.jpg"
date:       2015-07-08 12:00:00
author:     "Christophe Estanol"
share:      True
comments:   True
category:   blog
---

In this post we'll take a look at making simple changes to the front end of Spree.
As much as you can you should intent override Spree front end without changing the underlying view files. This will enable you to upgrade to newest version of Spree without headache. However there are situations where a direct override will be a lot easier.

### Customizing Spree views

Spree use [Deface](https://github.com/spree/deface) to make change to the views while keeping future upgrade . *Deface is a library that allows you to customize HTML (ERB, Haml and Slim) views in a Rails application without editing the underlying view.*

Changing the number of column from 3 to 2 using bootstrap grid classes looks like the following:

```ruby
Deface::Override.new(virtual_path: 'spree/products/show',
        name: 'columns-change-4-to-6',
        set_attributes: '.col-md-4',
        attributes: {class: 'col-md-6'})
```
Or removing the sidebar:

```ruby
Deface::Override.new(:virtual_path  => 'spree/products/index',
        name: 'landing-sidebar',
        remove: '[data-hook="homepage_sidebar_navigation"]')
```

### Responsive Header (nav-bar)

As we saw we can use deface to transform our header, we can for example:

remove the search-bar & main-navbar

```ruby
Deface::Override.new(virtual_path: 'spree/shared/_nav_bar',		
        name: 'remove-search-bar',		
        remove: '#search-bar')		

Deface::Override.new(virtual_path: 'spree/shared/_main_nav_bar',
        name: 'remove-main-nav',		
        remove: '#main-nav-bar')
```

Latest releases of Spree have included [Bootstrap](https://getbootstrap.com) for the front end to replace the previous Skeleton framework. The idea was to make upgrades slightly easier using the very popular Bootstrap markup. Spree will benefit from Bootstrap responsive best practices and improve usability and mobile support.
However Spree basic version doesn't collapse the nav-bar below 750px. Integrating this functionality with all the different partials and Deface seemed a bit complicated whereas creating the file and overriding it directly would save me a lot of time so I went for this option.

I just had to create the following file

 ***app/views/spree/shared/_header.html.erb***

```rhtml
<div id="spree-header">
  <header id="header" data-hook>

    <nav  class="navbar navbar-default" role="navigation">
      <div class="container-fluid">
        <!-- Brand and toggle get grouped for better mobile display -->
        <div class="navbar-header">
          <button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <figure id="logo"><%= logo %></figure>
        </div>
        <!-- Collect the nav links, forms, and other content for toggling -->
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
            <ul class="nav navbar-nav navbar-right" data-hook>
              <!-- if spree_current_user & spree_current_user.is_admin? -->
              <% if try_spree_current_user.try(:has_spree_role?, 'admin') %>
                <li id='admin-link' data-hook><%= link_to 'Admin', spree.admin_path %></li>
              <% end %>
              <% if spree_current_user %>
                <li><%= link_to Spree.t(:my_account), spree.account_path %></li>
                <li><%= link_to Spree.t(:logout), spree.logout_path %></li>
              <% else %>
                <li id="link-to-login"><%= link_to Spree.t(:login), spree.login_path %></li>
              <% end %>
              <li id="link-to-cart" data-hook>
                <noscript>
                  <%= link_to Spree.t(:cart), '/cart' %>
                </noscript>
                &nbsp;
              </li>
              <script>Spree.fetch_cart()</script>
            </ul>
        </div>
        <!-- /.navbar-collapse -->
      </div>
      <!-- /.container -->
    </nav>

 </header>
</div>
```

### Custom landing page

The way to do it is to create a new file ***app/views/layout/landing.html.erb***

```rhtml
<!doctype html>
<!--[if lt IE 7 ]> <html class="ie ie6" lang="<%= I18n.locale %>"> <![endif]-->
<!--[if IE 7 ]>    <html class="ie ie7" lang="<%= I18n.locale %>"> <![endif]-->
<!--[if IE 8 ]>    <html class="ie ie8" lang="<%= I18n.locale %>"> <![endif]-->
<!--[if IE 9 ]>    <html class="ie ie9" lang="<%= I18n.locale %>"> <![endif]-->
<!--[if gt IE 9]><!--><html lang="<%= I18n.locale %>"><!--<![endif]-->
  <head data-hook="inside_head">
    <%= render partial: 'spree/shared/head' %>
  </head>
  <body class="<%= body_class %>" id="<%= @body_id || 'default' %>" data-hook="body">
    <%= render partial: 'spree/shared/google_analytics.js' %>
    <%= render partial: 'spree/shared/header' %>

        <div id="content" data-hook>
          <%= flash_messages %>
          <%= yield %>
        </div>

        <%= yield :templates %>
    <%= render partial: 'footer/footer' %>
  </body>
</html>
```
(I have also created a partial for a footer as Spree doesn't include it in latest releases.)

And override the **HomeController** to include `layout 'landing'`
***app/controllers/spree/home_controller_decorator.rb***

```ruby
Spree::HomeController.class_eval do
 layout 'landing'
end
```
You also have to override the landing page with deface:
***app/overrides/landing.rb***

```ruby
Deface::Override.new(virtual_path: 'spree/home/index',
        replace: '[data-hook="homepage_products"]',
        name: 'landing-products',
        partial: 'home/landing',
        original: '461aae32b5912b8551fcf3a823427507f434a0cc')
```

And finally create ***app/views/home/_landing.html.erb***

In this example will use bootstrap carousel:

```rhtml
<div id="carousel-landing" class="carousel slide" data-ride="carousel">

  <!-- Indicators -->
  <ol class="carousel-indicators">
    <li data-target="#carousel-landing" data-slide-to="0" class="active"></li>
    <li data-target="#carousel-landing" data-slide-to="1"></li>
  </ol>

  <!-- Wrapper for slides -->
  <div class="carousel-inner" role="listbox">
    <div class="item active">
      <div class="container">
        <div class="row">
          <div class="col-md-7 carousel-img">

          </div>
          <div class="col-md-5 carousel-text">
            <h1>Lorem ipsum</h1>
            <h3>Lorem ipsum dolor sit amet</h3>
            <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem odit eveniet, fugiat inventore quo, exercitationem explicabo voluptas odio, tempore quibusdam dicta repellendus. Soluta corporis, maxime dicta, autem sit praesentium nulla?</p>
          </div>
        </div>
      </div>
    </div>
    <div class="item">
      <div class="container">
        <div class="row">
          <div class="col-md-7 carousel-img">

          </div>
          <div class="col-md-5 carousel-text">
            <h1>Lorem ipsum</h1>
            <h3>Lorem ipsum dolor sit amet</h3>
            <p>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Rem odit eveniet, fugiat inventore quo, exercitationem explicabo voluptas odio, tempore quibusdam dicta repellendus. Soluta corporis, maxime dicta, autem sit praesentium nulla?</p>
          </div>
        </div>
      </div>
    </div>
    <!-- End slides -->
  </div>
  <!-- End wrapper for slides -->
</div>
<!-- End carousel -->

<div class="container">
  <div class="row">
    <h3 class="text-center">Lorem ipsum</h3>
    <% if params[:keywords] %>
      <div data-hook="search_results">
        <% if @products.empty? %>
          <h6 class="search-results-title"><%= Spree.t(:no_products_found) %></h6>
        <% else %>
          <%= render :partial => 'spree/shared/products', :locals => { :products => @products, :taxon => @taxon } %>
        <% end %>
      </div>
    <% else %>
      <div data-hook="homepage_products">
        <% cache(cache_key_for_products) do %>
          <%= render :partial => 'spree/shared/products', :locals => { :products => @products, :taxon => @taxon } %>
        <% end %>
      </div>
    <% end %>
  </div>
</div>
```

With this simple modifications we can transform Spree front end and
