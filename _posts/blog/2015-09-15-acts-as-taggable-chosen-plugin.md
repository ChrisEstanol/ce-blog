---
layout:     post
title:      "Rails - Tagging with autocomplete"
subtitle:   "How to integrate acts_as_taggable gem with Chosen jQuery library for autocomplete"
header-img: "img/drjekyll.jpg"
thumb-img:  "img/drjekyll-thumb.jpg"
date:       2015-09-15 12:00:00
author:     "Christophe Estanol"
share:      True
comments:   True
category:   blog
---

For a project I was recently working on I had to set up a simple tagging system along with autocompletion. The idea was to guide the user towards a pre-defined list of tags that could be easily filtered. Amongst many options I opted for a great gem called [act_as_taggable](https://github.com/mbleigh/acts-as-taggable-on) and the [chosen](https://github.com/harvesthq/chosen) jQuery library.

For this example will be creating a simple application that manages products, this could be the base for a store for example. You can find this [example](https://github.com/ChrisEstanol/tagging_autocomplete) in my github.

### Let's start by creating our new application

```bash
rails new tagging_autocomplete -T
```
**-T** will omit all the test framework.

Then let's add our gems

```ruby
gem 'bootstrap-sass'
gem 'acts-as-taggable-on'
```

One way to add external JQuery plugins to your Rails app is to use [Rails Assets](https://rails-assets.org/). *It automatically converts the packaged components into gems that are easily droppable into your asset pipeline and stay up to date*. I came across Rails assets in this [Go Rails](https://gorails.com/episodes/rails-assets) episode an I would recommend to use it every time you need to include external JS libraries into your rails application.

Beside adding in your Gemfile

```ruby
source 'https://rails-assets.org' do
  gem 'rails-assets-chosen'
end
```
 you also need to:

```
Include following in application.js:
//= require chosen
Include following in application.css:
*= require chosen
```
And run `bundle install`.

Next we'll need to create our CRUD to manage products and tags

```bash
rails g scaffold Product name:string description:text
rails g scaffold Tag name:string
```

To create a leaner Scaffold without helpers, test, or assets you can:

* add `--no-helper --no-assets --no-controller-specs --no-view-specs --no-test-framework` after your scaffold command
* or include in **config/application.rb**

```ruby
config.generators do |g|
  g.assets            false
  g.helper            false
  g.test_framework    nil
  g.jbuilder          false
end
```

### Set up act_as_taggable gem

```bash
rake acts_as_taggable_on_engine:install:migrations
```
Review the generated migrations, look for the `ActsAsTaggableOnMigration` and remove

```ruby
create_table :tags do |t|
  t.string :name
end
```
As we already have created this table.

then run the migration :

```bash
rake db:migrate
```

Now we need to change in our **app/controllers/product_controller.rb**

The index action:

```ruby
def index
  if params[:tag]
    @products = Product.tagged_with(params[:tag])
  else
    @products = Product.all
  end
end
```
As well as the product_params:

```ruby
def product_params
  params.require(:product).permit(:name, :description, :tag_list)
end
```

In our **app/models/product.rb** we have to include

```ruby
acts_as_taggable
```

And in **app/views/products/_form.html.erb** add the tag_list field:

```rhtml
<%= f.label :tag_list, 'Tags (separated with comas)' %>
<%= f.text_field :tag_list, class: 'form-control' %>
```

To see your tag list add `<%= product.tag_list %>` in **/views/products/index.html.erb** and **/views/products/show.html.erb**.
At this stage we have built a simple tagging system but let's improve it a little with a way for the user to filter results.

### Filtering results by tag

As mentioned in this [Railscast](http://railscasts.com/episodes/382-tagging) acts_as_taggable documentation tells us that `tag_list` returns an array.

```ruby
@product.tag_list # => ["north", "east", "south", "west"]
```

So to create the filter link we just have to transform our **/views/products/index.html.erb** mapping each element of the array like this:

```rhtml
<%= raw product.tag_list.map { |t| link_to t, tag_path(t) }.join(', ') %>
```

To finish off the filtering functionality we need to tweak a little our routes to show the corresponding products when clicking on a tag and not trigger the show action.

```ruby
Rails.application.routes.draw do
  resources :tags, except: :show
  get 'tags/:tag', to: 'products#index'
  resources :products
  root 'products#index'
end
```

Now when you click on an individual tag you get a filtered result!

### Let's begin autocompletion

We are just a few steps away to finish with our application. For the autocompletion the Chosen library will do the heavy lifting, we just have to implement it.

One important thing we forgot when creating our migrations for the tagging was to set up our relations within the tag model so let's fix that in **app/model/tag.rb**:

```Ruby
class Tag < ActiveRecord::Base
  has_many :taggings
  has_many :tags, through: :taggings
end
```

We also need to permit the `tag_ids` parameters in our product_params:

```Ruby
def product_params
  params.require(:product).permit(:name, :description, :tag_list, :tag, { tag_ids: [] }, :tag_ids)
end
```

Change our `text_field` for a `collection_select` in our **views/products/_form.html.erb**:

```rhtml
<%= f.label :tag_list, "Tags" %>
<%= f.collection_select :tag_ids, Tag.order(:name), :id, :name, {}, {multiple: true} %>
```

And finally initiate chosen.js in **app/views/layouts/application.html.erb**. If we inspect the select attribute in the browser we'll see that rails create `id="product_tag_ids"`:

```js
  $(document).on('ready page:load', function () {
    $('#product_tag_ids').chosen({
      allow_single_deselect: true,
      width: '100%'
    })
  });
```

That's all, we made it and we have now created a tagging system with autocompletion. The source code for this application is in my [github repository](https://github.com/ChrisEstanol/tagging_autocomplete).
