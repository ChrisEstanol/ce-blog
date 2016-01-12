---
layout:     post
title:      "Rails - Ajax nested tables"
subtitle:   "Create dynamic nested tables with Ajax using Rspec for testing"
header-img: "img/tag.jpg"
thumb-img:  "img/tag-thumb.jpg"
date:       2015-10-12 12:00:00
author:     "Christophe Estanol"
share:      True
comments:   True
category:   blog
---

In this post we'll look into creating a simple application with nested tables. Nested tables allow to display additional information for nested sets of data (also known as trees or hierarchies) for relational databases. You could also apply this to any hierarchy based data format like MongoDB or simple JSON.
For this example we'll be showing the population of some european countries and cities to show the model relation `country has_many :cities`.
This type of functionality is inspired from the popular Jquery plugin [Data table](https://datatables.net/examples/api/row_details.html) to add **advanced interaction controls to any HTML table**.
We'll set up and use RSpec for our testing need.

### Why am I bothering with testing?

Becoming a better Rails developer will inevitably involve TDD and any leading Rails shop will heavily rely on testing to create and maintain error free code base. Testing as always been a part of coding but the Ruby community has embraced Automatic testing like any other. Testing establish additional trust when pushing your code to a team based project. It's simply about accountability. Also if you want to contribute to open source as some point you will need to provide your own tests.
We'll talk about testing in the future but for now let's build an app.

There are 2 main testing framework when working Ruby or Rails code:

* **[Minitest](https://github.com/seattlerb/minitest)**
 comes packaged with Ruby itself and provide a quick way to `assert` that your code won't fail. You can learn a lot about Minitest while reading Micahel Hartl [Rails tutorial](https://www.railstutorial.org/book).
* **[RSpec](https://github.com/seattlerb/minitest)**
 is a "Behaviour Driven Development for Ruby. Making TDD Productive and Fun". Well I couldn't agree more with the RSpec catch phrase. This framework is trying to solve something important about testing: **It should be more about your code than the test itself**.

 Some developer would also argue that you shouldn't be introducing a new language to test and you should be making your test using pure Ruby. I find that RSpec syntax is not too difficult to learn and make your tests more maintainable. Anybody that doesn't know your app can understand what it `should` `expect` from your code.

### Setting up RSpec

I have been reading [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec) and found it provides a clear and concise introduction to RSpec so you might recognize the familiar set up.

First thing first our new application:
```bash
rails new ajax_nested_table -T
```
**-T** will omit all the test framework as we are using RSpec.

Then let's add our gems and run `bundle`.

```ruby
group :development,:test do
  gem 'byebug'
  gem 'web-console', '~> 2.0'
  gem 'rspec-rails', '~> 3.1.0'
  gem 'factory_girl_rails', '~> 4.4.1'
 end

 group :test do
  gem 'faker', '~> 1.4.3'
  gem 'capybara', '~> 2.4.3'
  gem 'database_cleaner', '~> 1.3.0'
  gem 'launchy', '~> 2.4.2'
  gem 'selenium-webdriver', '~> 2.43.0'
 end
```

Now that we have all the gems needed to test with RSpec, let's install it with all the necessary structure:

```bash
rails generate rspec:install

```
Then include in **config/application.rb**

```ruby
config.generators do |g|
  g.assets            false
  g.helper            false
  g.test_framework :rspec,
    fixtures: true,
    view_specs: false,
    helper_specs: false,
    routing_specs: false,
    controller_specs: true,
    request_specs: false
  g.fixture_replacement :factory_girl, dir: "spec/factories"
  g.jbuilder          false
end
```
Note that I am voluntarily omitting **assets, helpers, test_framework and jbuilder** in this config.

Finally, let’s install a binstub for RSpec:

```bash
bundle binstubs rspec-core
```

This will create an RSpec executable, inside the application’s bin directory.

At his stage we are all set with our testing framework and if you run

```bash
rspec
```
You'll get:

```bash
No examples found.

Finished in 0.00027 seconds (files took 0.07494 seconds to load)
0 examples, 0 failuresIf we run
```

### Testing our models

As mentioned above our app will show the population of some european countries and cities so let's first get our models:

```bash
rails g model Country name:string population:integer
rails g model City name:string population:integer country:references
```
And run

```bash
rake db:migrate
```

rails generator while creating our schema and models also added new files to spec:

```
spec
  ├── factories
  ├── models
  |     ├── country_spec.rb
  |     └── city_spec.rb
```
RSpec will know what file to test by using the following convention `<<class name>>_spec.rb`.

We'll create our first test for our country model **spec/models/country_spec.rb**:

```ruby
require 'rails_helper'

describe Country do
  it "is valid with a name and a population number" do
    country = Country.new
    country.name = "France"
    country.population = 65447374
    expect(country).to be_valid
  end
  it "is invalid without a name" do
    country = Country.new(name: nil)
    country.valid?
    expect(country.errors[:name]).to include("can't be blank")
  end
  it "is invalid without a population number" do
    country = Country.new(population: nil)
    country.valid?
    expect(country.errors[:population]).to include("can't be blank")
  end
end
```

And to pass it we'll need to add some validation to **models/country.rb**

```ruby
class Country < ActiveRecord::Base
  has_many :cities
  validates :name, :population, presence: true
end
```

Now if we run:

```bash
rspec spec/models/country_spec.rb
```

We'll pass our firt test!!

```bash
Finished in 0.30295 seconds (files took 2.42 seconds to load)
3 examples, 0 failures
```
And to test our city model **spec/models/country_spec.rb** we'll do:

```ruby
describe City do
  it "is valid with a name and a population number" do
    france = Country.create(name: 'France', population: 65447374)
    city = City.new(country_id: france.id)
    city.name = "Paris"
    city.population = 2240622
    expect(city).to be_valid
  end
  it "is invalid without a name" do
    city = City.new(name: nil)
    city.valid?
    expect(city.errors[:name]).to include("can't be blank")
  end
  it "is invalid without a population number" do
    city = City.new(population: nil)
    city.valid?
    expect(city.errors[:population]).to include("can't be blank")
  end
end
```

We can nowrun:

```bash
rspec
```

And see that our all test are passing!

### Refactoring models tests with Factory Girl

One of the gem that we installed along with RSpec is called [factory_girl_rails](https://github.com/thoughtbot/factory_girl_rails) and it simply creates data to test our app.
When we generated our models earlier it automatically created a set of files in the factories folder.

```
spec
  ├── factories
  |     ├── cities.rb
  |     └── countries.rb
  ├── models
```

Here is how we set up `countries.rb`:

```ruby
FactoryGirl.define do
  factory :city do
    association :country
    name { Faker::Address.country }
    population { Faker::Number.number(7) }
  end
end
```
We'll also use [Faker](https://github.com/stympy/faker) to generate realistic dummy data so we don't have to.
And we can refactor our tests to a simpler:

```ruby
describe City do
  it "is valid with a name and a population number" do
    city = build(:city)
    expect(city).to be_valid
  end
```

One little config trick:

```ruby
# Include Factory Girl syntax to simplify calls to factories
  config.include FactoryGirl::Syntax::Methods
```

This line just simplify writing our test as we don't need to add `Factory.build` every time we set up a data object.
