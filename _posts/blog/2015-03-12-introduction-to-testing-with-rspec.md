---
layout:     post
title:      "Rails - Introduction to testing with RSpec"
subtitle:   "Implementing TDD with Ajax nested tables (RSpec, Capybara, Selenium & Factorygirl)"
header-img: "img//post/test.jpg"
thumb-img:  "img/post/thumb/test-thumb.jpg"
date:       2016-03-12 12:00:00
author:     "Christophe Estanol"
share:      True
comments:   True
category:   blog
sitemap:
  lastmod: 2016-06-19
  priority: 0.7
  changefreq: 'monthly'
---

In this post we'll look into creating a simple application with nested table. Nested tables allow to display additional information for nested sets of data (also known as trees or hierarchies) for relational databases. You could also apply this to any hierarchy based data format like MongoDB or simple JSON.
For this example we'll be showing the population of some european countries and cities to show the model relation `country has_many :cities`.
This type of functionality is inspired from the popular Jquery plugin [Data table](https://datatables.net/examples/api/row_details.html).

<img src="{{ site.baseurl }}/img/post/ajax-nested-table.gif" alt="Ajax nested table">

I won't be getting into too much details about the app itself and we would be focusing on the test coverage. If you want to have a look at the complete app you can check out the [repository on github](https://github.com/ChrisEstanol/ant).

### Why bothering with testing?

Becoming a better developer will inevitably involve TDD (Test Driven Development) and any experienced Rails developer will heavily rely on testing to create and maintain error free code base. Testing as always been a part of coding but the Ruby community has embraced Automatic testing like any other. Testing establish additional trust when pushing your code to a team based project. It's simply about accountability. Also if you want to contribute to open source as some point you will need to provide your own tests.

There are 2 main testing framework when working Ruby or Rails code:

* **[Minitest](https://github.com/seattlerb/minitest)**
 comes packaged with Ruby itself and provide a quick way to `assert` that your code won't fail. You can learn a lot about Minitest while reading Micahel Hartl [Rails tutorial](https://www.railstutorial.org/book).
* **[RSpec](https://github.com/seattlerb/minitest)**
 is a "Behaviour Driven Development for Ruby. Making TDD Productive and Fun". Well I couldn't agree more with the RSpec catch phrase. This framework is trying to solve something important about testing: **It should be more about your code than the test itself**.

 Some developer would also argue that you shouldn't be introducing a new language to test and you should be making your test using pure Ruby. I find that RSpec syntax is not difficult to learn and make your tests more maintainable. Anybody that doesn't know your app can understand what it `should` `expect` from your code.

### Setting up RSpec

I have been reading [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec) and found it provides a clear and concise introduction to RSpec so you might recognize the set up.

First thing first our new application:

```bash
rails new ajax_nested_table -T
```
**-T** will omit all the test framework as we are using RSpec.

Then let's add our gems and run `bundle`. In the example below I have omitted the gems version number but you should check them individually on the [rubygems.org](https://rubygems.org/) website and lock them at their current stable version to avoid gem inconsistencies in the future.

```ruby
group :development,:test do
  gem 'rspec-rails'
  gem 'factory_girl_rails'
 end

 group :test do
  gem 'faker'
  gem 'shoulda-matchers'
  gem 'capybara'
  gem 'database_cleaner'
  gem 'launchy'
  gem 'selenium-webdriver'
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
0 examples, 0 failures
```

### Testing our models

As mentioned above our app will show the population of some european countries and cities so let's first create our models:

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

Let's create our first test for our country model **spec/models/country_spec.rb**. We'll create a basic evaluation of our form validation.

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

ANd if we run `rspec` in our terminal the result is obviously a series of failed examples and one pending (our city spec that was created when running rails generator).

```bash
Finished in 0.01149 seconds (files took 2.92 seconds to load)
4 examples, 2 failures, 1 pending

Failed examples:

rspec ./spec/models/country_spec.rb:10 # Country is invalid without a name
rspec ./spec/models/country_spec.rb:15 # Country is invalid without a population number
```

And to pass this set of tests we'll need to add some validation to **models/country.rb**

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
The process is pretty similar to test our city model **spec/models/city_spec.rb** we'll do:

```ruby
require 'rails_helper'

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

We can now run:

```bash
rspec
```

And see that our test are failing :-(. Not to worry because to pass we just need to add simple validation like for our **models/country.rb**.


```ruby
class City < ActiveRecord::Base
  belongs_to :country
  validates :name, :population, presence: true
end
```

### Refactoring models tests

#### Factory Girl

One of the gem that we installed along with RSpec is called [factory_girl_rails](https://github.com/thoughtbot/factory_girl_rails) and it simply creates data to test our app.
When we generated our models earlier it automatically created a set of files in the factories folder.

```
spec
  ├── factories
  |     ├── cities.rb
  |     └── countries.rb
  ├── models
```

Here is how we set up **spec/factories/cities.rb**:

```ruby
FactoryGirl.define do
  factory :city do
    association :country
    name { Faker::Address.country }
    population { Faker::Number.number(7) }
  end
end
```
And **spec/factories/countries.rb**:

```ruby
FactoryGirl.define do
  factory :country do
    name { Faker::Address.country }
    population { Faker::Number.number(7) }
  end
end
```

We'll also use [Faker](https://github.com/stympy/faker) to generate realistic dummy data so we don't have to.

One little config trick to add to **rails_helper.rb**:

```ruby
# Include Factory Girl syntax to simplify calls to factories
  config.include FactoryGirl::Syntax::Methods
```

This line just simplify writing our test as we don't need to add `Factory.build` or `Factory.create` or `FactoryGirl.attributes_for` every time we set up a data object.

And we can refactor our tests to a simpler:

```ruby
require 'rails_helper'

describe Country do  

  describe "validations" do
    it "is valid with a name and a population number" do
      country = build(:country)
      expect(country).to be_valid
    end
    it "is invalid without a name" do
      country = build(:country, name: nil)
      country.valid?
      expect(country.errors[:name]).to include("can't be blank")
    end
    it "is invalid without a population number" do
      country = build(:country, population: nil)
      country.valid?
      expect(country.errors[:population]).to include("can't be blank")
    end
  end

  describe "associations" do
    it "has many cities" do
      assc = described_class.reflect_on_association(:cities)
      expect(assc.macro).to eq :has_many
    end
  end

  it "has a valid factory" do
    expect(build(:country)).to be_valid
  end

end
```

We also have added another test to verify our associations using ActiveRecord [reflect_on_association](http://apidock.com/rails/v2.3.8/ActiveRecord/Reflection/ClassMethods/reflect_on_association).


#### Shoulda-matchers

Another way to refactor our models specs would be to use the `shoulda-matchers gem`.
It would simplify our **country_spec.rb** to:

```ruby
require 'rails_helper'

describe Country do
  context 'associations' do
    it { should have_many(:cities) }
  end

  context 'validations' do
    it { should validate_presence_of :name }
    it { should validate_presence_of :population }
  end

  it "has a valid factory" do
    expect(build(:country)).to be_valid
  end
end
```

You can take a look at the [sample app](http) on github to see how to refactor the **city_spec.rb**. Again it doesn't really change much from the country model you just need to assess the presence of the ActiveRecord `belongs_to` association.

```ruby
describe City do

  describe "validations" do
    it {should belong_to(:country)}
  end

  ...
```

And that's about it for our models. With the help of Factorygirl, Fakers and Shoulda-matchers we have refactored our tests to cover our models validations and associations using just a few lines of code. It's time now to move on  to test our controllers.

### Controllers specs

When starting with the task of testing your controller you should generate a scaffold to see how Rspec generator typically create a default test. Once your application is correctly set up with Rspec, generating models, controllers or scaffolds from the command line will also create associated specs.

We can run `rails g scaffold Country name population:integer --skip` and have a look at the generated files. Just make sure to use `--skip` so we are not overwriting the model **models/country.rb**,  the associated specs **specs/moodels/country_spec.rb** and factories **spec/factories/countries.rb** as we just set them up.

The basic syntax of a controller spec:

We use a `describe` block to describe each controller actions, that by convention as instance methods are prefixed with a hash symbol. We also indicate the corresponding HTTP verb. Then we use an `it` block for the test case.

We will be making small changes to the default spec like including Factorygirl `attributes_for` or `create(:country)` to simplify it, the rest is pretty much standard.
We'll also create an `:invalid_country` in our factory **factories/country.rb** to test when passing invalid attributes.

```ruby
factory :invalid_country do
  name nil
end
```

Here is our complete controller spec for `CountriesController`.

```ruby
require'rails_helper'

describe CountriesController do

  describe 'GET #index' do
    it "populates an array of all countries" do
      france = create(:country, name: "France", population: 60000000)
      germany = create(:country, name: "Germany", population: 80000000)
      get :index
      expect(assigns(:countries)).to match_array([france, germany])
    end

    it "renders the :index template" do
      get :index
      expect(response).to render_template :index
    end
  end

  describe 'GET #show' do
    it "assigns the requested country to @country" do
      country = create(:country)
      get :show, id: country
      expect(assigns(:country)).to eq country
    end

    it "renders the :show template" do
      country = create(:country)
      get :show, id: country
      expect(response).to render_template :show
    end
  end

  describe 'GET #new' do
    it "assigns a new country to @country" do
      get :new
      expect(assigns(:country)).to be_a_new(Country)
    end
    it "renders the :new template" do
      get :new
      expect(response).to render_template :new
    end
  end

  describe 'GET #edit' do
    it "assigns the requested country to @country" do
      country = create(:country)
      get :edit, id: country
      expect(assigns(:country)).to eq country
    end

    it "renders the :edit template" do
      country = create(:country)
      get :edit, id: country
      expect(response).to render_template :edit
    end
  end

  describe 'POST #create' do
    context "with valid attributes" do
      it "saves the new country in the database" do
        expect{
          post :create, country: attributes_for(:country)
        }.to change(Country, :count).by(1)
      end
      it "redirects to countries#show" do
        post :create, country: attributes_for(:country)
        expect(response).to redirect_to country_path(assigns[:country])
      end
    end

    context "with invalid attributes" do
      it "does not save the new country in the database" do
        post :create, country: attributes_for(:country, name: nil)
        expect(Country.count).to eq(0)
      end
      it "re-renders the :new template" do
        post :create, country: attributes_for(:country, name: nil)
        expect(response).to render_template :new
      end
    end
  end

  describe 'PATCH #update' do
    before :each do
      @country = create(:country, name: 'Germany', population: 80000000)
    end

    context "with valid attributes" do
      it "locates the requested @country" do
        patch :update, id: @country, country: attributes_for(:country)
        expect(assigns(:country)).to eq(@country)
      end
      it "changes @country's attributes" do
        patch :update, id: @country,
          country: attributes_for(:country,
            name: 'Switzerland',
            population: 8000000)
        @country.reload
        expect(@country.name).to eq('Switzerland')
        expect(@country.population).to eq(8000000)
      end
      it "redirects to the updated country" do
        patch :update, id: @country,
          country: attributes_for(:country)
        expect(response).to redirect_to @country
      end
    end

    context "with invalid attributes" do
      it "does not change the country's attributes" do
        patch :update, id: @country,
          country: attributes_for(:country,
            name: 'Swizterland',
            population: nil)
        @country.reload
        expect(@country.name).not_to eq('Swizterland')
        expect(@country.population).to eq(80000000)
      end
      it "re-renders the :edit template" do
        patch :update, id: @country,
          country: attributes_for(:country, name: nil)
        expect(response).to render_template :edit
      end
    end
  end

  describe 'DELETE #destroy' do
    before :each do
      @country = create(:country)
    end

    it "deletes the country from the database" do
      expect{
        delete :destroy, id: @country
      }.to change(Country,:count).by(-1)
    end

    it "redirects to countries#index" do
      delete :destroy, id: @country
      expect(response).to redirect_to countries_path
    end
  end
end

```

### Features specs

We have now spent a good amount of time installing and configuring Rspec and created a series of unit tests for our models and controllers. It's time now to integrate features tests or as it is sometimes called acceptance tests.

This is the favorite part of testing for Rails developers as we get to test the real functionalities and rendering of the app. We will be using [Capybara](http://jnicklas.github.io/capybara/) to simulate real-world use of the application.

#### capybara

So as mentioned Capybara lets you simulate how a user would interact with your application through a web browser, using a series of easy to understand methods like `click_link`, `fill_in`, and `visit`.

Let's practice with our first test to check that our user is visiting our homepage.
In our spec folder we create a features folder with a file called `user_visits_homepage_spec.rb`.

```
spec
  ├── features
  |     └── user_visits_homepage_spec.rb
  ├── models
```

```ruby
require "rails_helper"

feature "User visits homepage" do
  scenario "successfully" do
    visit root_path

    expect(page).to have_css 'h1'
    expect(page).to have_content('Listing Countries')
  end
end
```

Capybara uses his own [DSL (Domain Specific Language)](https://en.wikipedia.org/wiki/Domain-specific_language) and introduce the notion of `feature` and `scenario` to define a test. You can also learn more about capybara helpers and methods like `have_css` and `have_content` in the [documentation](http://www.rubydoc.info/github/jnicklas/capybara/master#The_DSL).

Let's now build an actual feature test and learn more about Capybara syntaxes. Can you guess what this feature spec does?

```
spec
  ├── features
  |     ├── user_creates_country_spec.rb
  |     └── user_visits_homepage_spec.rb
  ├── models
```

```ruby
require "rails_helper"

feature "Countries management" do
  scenario "User creates country" do
    visit root_path
    expect{
      click_link "New Country"
      fill_in "Name", with: "Colombia"
      fill_in "Population", with: "60000000"
      click_button "Create Country"
    }.to change(Country, :count).by(1)
    country = Country.last
    expect(country).not_to be_nil
    expect(current_path).to eq country_path(country)
    expect(page).to have_content "Country was successfully created."
    visit root_path
    expect(page).to have_css 'table td', text: "Colombia"
  end
end
```

As you may have guessed we have implemented a test `user_creates_country_spec.rb` that covers the creation of a new country. It is pretty self explanatory, we are simulating a series of events and expecting to find the country created in a table in our root page.
We can run `rspec` and watch the test fail and then implement the solution.

To make the test pass we'll have to assign the `root` of our app to our `countries#index` in our **config/routes.rb** file.


We can also use a similar test to cover the creation of a city **user_creates_city_spec.rb**, with the slight difference that when filling the form we need to select the Country where the city belongs.

```ruby
require "rails_helper"

feature "Cities management" do
  let!(:country) do
    create(:country, name: 'Colombia', population: 60000000)
  end

  scenario "User creates city" do
    visit root_path
    expect{
      click_link 'New City'
      select 'Colombia', from: 'Country'
      fill_in 'Name', with: 'Bogota'
      fill_in 'Population', with: '60000000'
      click_button 'Create City'
    }.to change(City, :count).by(1)
    city = City.last
    expect(city).not_to be_nil
    expect(current_path).to eq city_path(city)
    expect(page).to have_content 'City was successfully created.'
  end
end
```

We have introduced a new syntax `let` that you will see a lot amongst projects using Rspec.
[Let](https://www.relishapp.com/rspec/rspec-core/v/2-5/docs/helper-methods/let-and-let) is one of many Rspec helpers that simplify writing test, provide best practice and reduce keystrokes.
Up until now we have used `:before` blocks to assign test data to instance variables.
In this particular case we'll use `let!` so that the value is persisted into the database. The one core difference to before blocks is that you get an explicit reference to this variable, rather than needing to fall back to instance variables.
Also let() is lazy-evaluated. This means that let() is not evaluated until the method that it defines is run for the first time.

To make **user_creates_city_spec.rb** pass we need first to add a `New City` button to our **countries/index.html.erb**:

```ruby rhtml
<%= link_to 'New City', new_city_path %>
```

And then we can use a scaffold `rails g scaffold City name population:integer --skip` to create the controller and the views. For now we can remove the file **cities_controller_spec.rb** that was automaticaly created.

To complete this step we will need to add a select field to create a City for a specific country to our **cities/_form.html.erb**

```rhtml
<div class="field">
  <%= f.label :country_id %>
  <%= f.collection_select :country_id, Country.all, :id, :name, {include_blank: 'Choose a country'} %>
</div>
```

And assign the `country_id` in the **cities_controller_spec.rb** `create` method

```ruby
@city.country_id = params[:city][:country_id]
```

### Four Phase Test Pattern

After going through these rspec examples, you may recognize a pattern for writing test. For implementing testing best practice you should follow the Four-Phase Test pattern:

```
test do
  # setup - Prepare object for the test
  # exercise - Execute the functionality we are testing  
  # verify - Verify the exercise's result against our expectation  
  # teardown - Resetting all data to pre-test state
end
```

### Testing with Javascript

We need to implement some changes in order to test our Ajax request and how we dynamically populate part of our tables. As you remember the whole purpose of this application is to fetch cities associated to a country through Ajax.
If your application or front-end rely heavily on Javascript you might need to use a specific testing framework like [Mocha or Jasmine](http://thejsguy.com/2015/01/12/jasmine-vs-mocha-chai-and-sinon.html) but in our case [Selenium web-driver](http://www.rubydoc.info/github/jnicklas/capybara/master#Selenium) will be just fine.
Although Capybara’s default `Rack::Test` driver does not support JavaScript, it bundles support for the Selenium web driver out of the box. With Selenium, you can simulate more complex web interactions, including JavaScript. Selenium makes this possible by running your test code through a lightweight web server, and automating the browser’s interactions with that server.
As you'll see when running your test it starts Firefox and run the Javascript interaction.

We also need to configure Database Cleaner to help with database transactions in our tests.

#### Database cleaner

Database Cleaner is a set of strategies for cleaning your database. You can find more information about use and config in the [documentation](https://github.com/DatabaseCleaner/database_cleaner).
You may want to read more about the strategies

We need to make the following adjustments to `spec/rails_helper.rb` to integrate the database_cleaner gem.

First switch this line to `false`.

```ruby
config.use_transactional_fixtures = false
```

And then paste the following configuration:

```ruby
  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end

  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end

  config.before(:each, :js => true) do
    DatabaseCleaner.strategy = :truncation
  end

  config.before(:each) do
    DatabaseCleaner.start
  end

  config.after(:each) do
    DatabaseCleaner.clean
  end
```

#### The final test

We'll create **features/user_toggles_city_list_spec.rb**

```ruby
require "rails_helper"

feature "Cities management" do
  let(:country) do
    create(:country, name: 'Colombia', population: 60000000)
  end
  let!(:city) do
    create(:city, country_id: country.id, name: 'Bogota', population: 60000000)
  end

  scenario "toggles city list", js: true do
    visit root_path
    find('a#Colombia').click
    expect(page).to have_css 'table td', text: 'Bogota'
  end
end
```

To enable selenium to run we'll add `js: true` to our scenario.

To make this test green we will implement our Ajax request.

First we'll define a custom route:

```ruby
get 'countries/cities/:id', to: 'countries#cities', as: 'countries_cities'
```

Then we create a method in our `CountriesController`:

```ruby
def cities
  @country = Country.find(params[:id])
  respond_to do |format|
    format.js
  end
end
```

As we specified `format.js` this method will look for a **views/countries/cities.js.erb** file. Let's create it.

```ruby
$('#cities_<%= @country.id %>').html("<%= j (render 'countries/cities') %>");
$('#cities_<%= @country.id %>').fadeToggle();
```

And we'll render a partial **views/countries/_cities.html.erb**.

```rhtml
<% @country.cities.each do |city| %>
  <tr class="cities">
    <td>   </td>
    <td><%= city.id %></td>
    <td><%= city.name %></td>
    <td><%= city.population %></td>
  </tr>
<% end %>
```

We will also make some change to the table in our **views/countries/index.html.erb** and use the `remote:` option.

```rhtml
<table class="table">
  <thead>
    <tr>
      <th></th>
      <th>id</th>
      <th>Name</th>
      <th>Population</th>
    </tr>
  </thead>
<% @countries.each do |country| %>
  <tbody>
      <tr>
        <td><%= link_to '+', countries_cities_path(country), remote: true, id: country.name %></td>
        <td><%= country.id %></td>
        <td><%= country.name %></td>
        <td><%= country.population %></td>
      </tr>
  </tbody>
  <tbody id='cities_<%= country.id %>' style='display: none'></tbody>
<% end %>
</table>
```

And that's it. All our test are now passing (green) and we have build a strong coverage of our application.

### Skipping controllers specs

Many experienced Rails developers would remove from their testing arsenal controllers specs altogether. Controllers specs AND feature specs will overlap and making sure that your app works in the browser with simulated user experience just makes more sense. This is where TDD (Test Driven Development) gets a bit of BDD (Behaviour Driven Development) and we extend testing to incorporate business requirements. If you work with Agile methodology and implement new features from users stories you can directly write the scenarios into your tests.

Getting an extensive test coverage is very important in our project development however if you have time only for one just focus on features and don't waste time writing controllers specs. In most cases, we don’t use controller specs unless there is complex logic inside the controller.
