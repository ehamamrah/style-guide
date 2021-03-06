Boundless Drop Ruby/Rails Style Guide:
======================================
This is Boundless Drop's Ruby Style Guide.

This was inspired by [bbatsov's](https://github.com/bbatsov/rails-style-guide) and [Airbnb's](https://github.com/airbnb/ruby) guides.

This is always going to be a work in progress and it is going to adapt to whatever is better suiting our team in terms of what will achieve better code quality and cleaner code.


------------------

1. [Clarity, Readability & Convention](#clarity-readability--convention)
2. [Database Level](#database-level)
3. [Models](#models)
4. [Views](#views)
5. [Controllers](#controllers)
6. [Routing](#routing)
7. [I18n](#i18n)
8. [Authorization](#authorization-with-cancancan)
9. [Gemfile](#gemfile)
10. [TODO](#todo)

Clarity, Readability & Convention:
----------------------------------
- Always use a Linter such as [rubocop](https://github.com/bbatsov/rubocop).
- Use the Syntastic plugin for vim to work with Rubocop.
- Methods should not be long than 8-9 lines. If they do, then you probably need to consider refactoring.
- Classes should not be longer than 100-120 lines. If they do, then remember SOLID prenciples and follow best practises.
- Use comments only if absolutely necessary and indent them with one space after the pound sign.
  ```ruby
  # Bad
  def list_all_results
    #get results
    Result.all
  end

  def initialize(options)
    #employee id required if only employee process
    @from_date = options[:from_date]
    @to_date   = options[:to_date]
    @employee_id = options[:employee_id]
  end

  # Good
  def list_all_results
    # Return ZERO is required if no params passed
    return 0 unless params.present?
    ...
  end

  def initialize(options)
    @from_date = options[:from_date]
    @to_date   = options[:to_date]
    @employee_id = options[:employee_id] # Required for employee transaction only.
  end

  ```
- Do not use Integers as keys for Hashes.
  ```ruby
  # Bad
  bad_hash = { 1: '',
               2: ''  }

  # Good
  good_hash = { first_element:  'Something',
                second_element: 'Something Else' }
  ```

- Hashes should always use symbols rather than strings for their keys
  ```ruby
  hash_example = { first_element:  'Example',
                   second_element: 'Another Example' }
  ```

- For active record where querying for an attribute in array pass it a hash instead of writing a string where query
- Prefer single-quoted strings when you don't need string interpolation or special symbols such as `'`.

- Defining and calling methods should use parenthesis
  ```ruby
  def hello(name)
    puts "#{name}"
  end

  hello('Sabri')
  ```
- Always use &&, ||, ! instead of language words like and, or, not
- Use CamelCase for classes and modules, snake_case for variables and methods, SCREAMING_SNAKE_CASE for constants.
  ```ruby
  # Good
  module ApplicableTransaction
    ...
  end

  class EmployeeTransaction < ActiveRecord::Base
    # Constants
    EMPLOYEE_STATUS = %i[Pending Activated Banned].freeze

    # Methods
    def list_employee_transaction; end
  end
  ```
- Break down long chain method calls for readability
- Methods that return boolean should end with a “?”
```ruby
def able_to_request_transaction?
  ...
end
```
- Methods that perform some permanent or potentially dangerous change should ending in "!"
```ruby
def send_urgent_request!
  ...
end
```

- Use Hash params instead of splat operator (*args)
  ```ruby
  def hello(options = {})
    first_name  = options[:first_name]
    middle_name = options[:middle_name]
    last_name   = options[:last_name]

    puts "Hello #{first_name} #{middle_name} #{last_name}"
  end

  hello(first_name: 'Sabri', middle_name: 'Saaber', last_name: 'Sabri')
  ```
- Use `each`, not `for`, for iteration.
- Prefer `private` over `protected` for non-public `attr_reader`s, `attr_writer`s, and `attr_accessor`s.

- Naming for methods should identify what it do.
  ```ruby
  # Bad
  def calculation; end

  # Good
  def calculate_employees_salaries; end
  ```
- Naming in Iteration should be clear and related to the iterated elements.
  ```ruby
  # Bad
  def list_employees_locations
    @employees.each { |e| ... }
  end

  # Good
  def list_employees_locations
    @employees.each { |employee| ... }
  end
  ```
- Indent when as deep as case.
  ```ruby
  case organization
  when 'Akhtaboot'
    puts 'akhtaboot.com'
  when 'ZenHR'
    puts 'zenhr.com'
  end

  # or
  workplace = case organization
              when 'Akhtaboot' then 'Near 5th circle'
              when 'ZenHR'     then 'Sweifieh'
  ```

- Align parameters in one line or one for each line.
  ```ruby
  def create_financial_list(employee,
                            branch,
                            from_date,
                            to_date)
    ...
  end

  ```

- Use spaces around operators; after commas, colons, and semicolons; and around { and before }.
  ```ruby
  total = price * rate
  total > item_value ? 'High Price' : 'Acceptable'
  ['ZenHR', 'Akhtaboot', 'Cavall', 'Testello'].each { |product| ... }
  ```

- Don't include spaces inside block parameter or arrays.
  ```ruby
  # Bad
  @branches_locations.each { | name, location | ... }

  # Good
  @branches_locations.each { |name, location| ... }

  # Bad
  [ 'Akhtaboot' , 'ZenHR' , 'Cavall' , 'Testello' ]

  # Good
  ['Akhtaboot', 'ZenHR', 'Cavall', 'Testello']
  ```

- Include newline between each method.
  ```ruby
  # Bad
  def list_of_locations; end
  def list_of_branches; end

  # Good
  def list_of_locations; end

  def list_of_branches; end
  ```

- Don't include newlines between areas that have a different indentations.
  ```ruby
  # Bad
  def list_of_active_users

    if user.active?
      ...
    end

    return ... if ...

  end

  # Good
  def list_of_active_users
    if user.active?
      ...
    end
    return ... if ...
  end
  ```

Database level:
--------------
- Never use #{variable} in any SQL queries for obvious security vulnerability
- While using Active Record methods such as `where` do not use `?`. Instead, use named arguements as follows:
```ruby
  # bad
  branch.taxes.where('start_date <= ? AND (end_date > ? OR end_date IS NULL)', date, date)

  # good
  branch.taxes.where('start_date <= :date AND (end_date > :date OR end_date IS NULL)', { date: date })
```

Models:
---------------------------
- Models are for business logic and data-persistence ONLY!
- Name models after nouns and not verbs
- Meaningful but short names
- Don't make it Fat, Consider Concerns & Design Patterns
- When defining model scopes, use a lambda to wrap the relation
```ruby
# Bad
scope :published, where(published: true)

# Good
scope :published, (-> { where(published: true) })
```
- Avoid return when it's not required
```ruby
# Bad
def list_employees_transactions
  return branch.employees.transactions
end

# Good
def list_employees_transactions
  branch.employees.transactions
end
```
- Use :: only to reference to constants, classes, modules or constructors.
```ruby
# Bad
Transaction::any_method
transaction_object::list_branch_transactions

# Good
Transaction.call_a_method
BranchTransaction::TRANSACTION_STATUSES
transaction_object.list_branch_transactions
```
- Feel free to introduce non ActiveRecord models. If validations are needed, use [ActiveAttr](https://github.com/cgriego/active_attr) gem.

The following model structure is to be used in all our models:

  ```ruby
  class User < ActiveRecord::Base
    include Something::Something

    # Constants
    ERRORS = { warning: 'Warning', danger: 'Danger' }.freeze

    # Associations
    has_one :life
    has_many :friends
    has_many :families
    belongs_to :mentor
    belongs_to :team

    # ActiveRecord Validations
    validates :name, presence: true
    validates :mentor_id, presence: true, if: :has_contact_with_mentor?
    validates :username, uniqueness: true, presence: true

    # Custom Validations
    validate :username_is_registered

    # Nested associations
    accepts_nested_attributes_for :something, reject_if: :all_blank, allow_destroy: true

    # Rails Delegations
    delegate :email, to: mentor, prefix: true, allow_nil: true
    delegate :first_name, to: mentor, prefix: true, allow_nil: true
    delegate :name, to: mentor, prefix: true, allow_nil: true

    # attr methods
    attr_accessor :something
    attr_writer :something_else
    attr_reader :something_different

    # ActiveRecord callbacks
    before_save :clean_username
    after_save :send_welcome_pack

    # scopes
    scope :published, (-> { where(published: true) })

    # class methods
    def self.i_am_a_class_method
      puts 'Hello, I am a class method!'
    end

    # Instance methods
    def i_am_an_instance_method
      puts 'Hello, I am an instance method!'
    end

    protected

    def i_am_a_protected_instance_method
      puts 'Hello, I am a protected instance method!'
    end

    private

    def i_am_a_private_instance_method
      puts 'Hello, I am a private instance method!'
    end

    # Custom validation methods
    def username_is_registered; end
  end
  ```

Views:
------
- Should contain as little logic as possible
- Logic should be handled elsewhere, preferably in Helpers.
```ruby
# Bad
= f.select :user_id, options_for_select(@site.users.map { |user| [user.name, user.id]}), ...

# Good
[View Level]
= f.select :user_id, options_for_select(active_site_users)
[Helper Level]
def active_site_users
  ...
end
```
- Don't use static translations, Define a translation key into our locales and call it.
```ruby
# Bad
= link_to t('create'), some_where_path, ...
= link_to 'Create', some_where_path, ...

# Good
= link_to t(:create), some_where_path, ...
```
Controllers:
------------
- Receive events from the outside world (usually through views)
- Interact with the model
- Displays the appropriate view to the user
- When it's becoming Fat, Consider using Concern Or Services.

The following structure is to be used in all our controllers:

  ```ruby
  class UsersController < ApplicationController
    def index; end

    def show; end

    def new; end

    def create; end

    def edit; end

    def update; end

    def destroy; end

    private

    def user_params
      params.require(:user).permit(:name, :email)
    end
  end
  ```

Routing:
-------
- Avoid the `:except` option in routes.
- Use the `:only` option to explicitly state exposed routes.
- Order resourceful routes alphabetically by name.
- Use nested routes to explain relation between models
- If you need to nest routes more than 1 level deep then use the shallow: true option. This will save user from long urls posts/1/comments/5/versions/7/edit and you from long url helpers edit_post_comment_version.

  ```ruby
  resources :posts, shallow: true do
    resources :comments do
      resources :versions
    end
  end
  ```

I18n:
----
- All strings that will be used in views MUST be written as a translation key using I18n's translate method.
- Translation keys needs to be added based on Alphabetical order.
- All translation keys should be in snake_case.
- All translation keys should use symbol syntax and not string syntax
- We use I18n translate as follows:
  ```ruby
  # In Models
  I18n.t(:translation_key)

  # Everywhere else
  t(:translation_key)
  ```
- All translation keys should be added to both en.yml and ar.yml which are located in config/locales/ .
- Custom validation errors should be written as follows:
  ```ruby
  I18n.t('model_name_errors.translation_key')
  ```
- Translations files should be structured as follows:
  ```yaml
  # Indentations are very important so be careful
  lang:
      # Normal Translations
      he_is_very_sabri: He is very Sabri
      i_like_trains: I like trains

      # Errors Translations
      model_name_errors:
          cant_be_this: "Can't be this"

      # Constants Translations
      people_categories:
          bad: Bad
          good: Good

      # Responses Translations
      some_responses:
          has_been_saved: Has been saved!

      # ActiveRecord Translations
      activerecord:

        # Models Translations
        models:
            another_model: Another Model
            model_name: Model Name

        # Attributes Translations
        attributes:
            model_name:
                another_one: Another One
                attribute: Attribute
  ```
- Translations inside each group should be sorted alphabetically.
- Gems translations should be in their own separate files in the locales directory and should be named as follows: gemname.lang.yml .
- Directory structure example:

  ```
  .
  ├── ar.yml
  ├── en.yml
  ├── paperclip.ar.yml
  └── paperclip.en.yml
  ```

Authorization with Cancancan:
----------------------------
In ZenHR we are using the gem [cancancan](https://github.com/CanCanCommunity/cancancan) to handle all authorization related matters.

A sample controller without using cancancan looks like so:

```ruby
class VacationsController < ApplicationController
  before_action :set_vacations

  def index
    @vacations = Vacation.all
  end

  def show
    @vacation
  end

  # etc.....

  private

  def set_vacations
    @vacation = Vacation.find(params[:id])
  end
end
```

As you can see there is no way to authorize different User roles to access/update/destroy the Vacation resources.

In comes cancancan:


```ruby
class VacationsController < ApplicationController
  load_and_authorize_resource

  def index; end

  def show; end
end
```

The `load_and_authorize_resource` callback is provided by the gem and it allows us to have access to the `@vacation` or `@vacations` depending on whether the action was a member ot collection resource.
Therefore there is no need for `set_vacations` anymore.

The `load_and_authorize_resource` is split into two parts:
- `load_resource`
- `authorize_resource`

The load part will give us access to the instance variable and the authorize part will handle the authorization rules from the `ability.rb` file.

Please take a look at the extensive documentation provided by cancancan to get a better understanding at how it works.

Gemfile:
----------------------------
- Always add necessary gems in Alphabetical order
- Group gems if it's required only in a level of use.
```ruby
group :development do
  gem 'better_errors'
  gem 'bullet'
  gem 'rubocop'
  gem 'spring'
end

group :development, :test do
  gem 'byebug', platform: :mri
  gem 'factory_bot_rails'
  gem 'faker', '~> 1.7.3'
  gem 'guard'
  gem 'guard-rspec', require: false
  gem 'guard-rubocop'
end
```

## TODO:
- How to deal with fat models
- Hash key and value naming
- Config section
- how to deal with env variables
- naming custom validation methods and how to deal with them
- naming variables and methods
- Null objects
- Making use of the rails CLI
- When to use constants
