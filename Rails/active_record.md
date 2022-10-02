# Active Record

## Active Record Basics

### Create

- `new` only instantiates, while `Create` instantiates and saves a record into databse
- Can pass new object to block for initialization

### Read

- user.[all, first, find_by{key: val}, where(hash)].order

### Update

- user.update(hash) to save right away

### Delete

- user.[destroy, destroy_by(hash), destroy_all]

### Validations

- validates :name, presence: true
- save returns false, while save! raises an exception

## Active Record Migrations

If the database transactions for schema-changing statements, migrations will be wrapped in a transaction.

If the database does not support such transactions, then when a migration fails, 
the parts that succeeded before failure will not be rolled back, you need to do this yourself.

### Telling AR how to reverse

- use reversible, obj.up, obj.down
- create up and down methods containing changes

### Creating a migration

#### Creating a Standalone Migration

```sh
rails generate migration AddPartNumberToProducts

# Adding multiple details
rails generate migration AddDetailsToProducts part_number:string price:decimal

# Reference (foreign key)
rails generate migration AddUserRefToProducts user:references
```

#### Model Generators

```sh
generate model Product name:string description:text
```

#### Passing Modifiers

```sh
rails generate migration AddDetailsToProducts 'price:decimal{5,2}' supplier:references{polymorphic}
```

### Writing a Migration

Happens after creating one using one of the methods above

#### Creating a table

```ruby
create_table :products do |t|
  t.string :name
end
```

#### Join table with indices

```ruby
create_join_table :products, :categories do |t|
  t.index :product_id
  t.index :category_id
end
```

#### Changing tables

```ruby
change_table :products do |t|
  t.remove :description, :name
  t.string :part_number
  t.index :part_number
  t.rename :upccode, :upc_code
end
```

#### Changing columns

```ruby
# Change column command is irreversible
change_column :products, :part_number, :text

# change name field to not null
change_column_null :products, :name, false

# change default value of the :approved field from true to false.
change_column_default :products, :approved, from: true, to: false
```

#### Column modifiers

- comment Adds a comment for the column.
- collation Specifies the collation for a string or text column.
- default Allows to set a default value on the column. Note that if you are using a dynamic value (such as a date), the default will only be calculated the first time (i.e. on the date the migration is applied). Use nil for NULL.
- limit Sets the maximum number of characters for a string column and the maximum number of bytes for text/binary/integer columns.
- null Allows or disallows NULL values in the column.
- precision Specifies the precision for decimal/numeric/datetime/time columns.
- scale Specifies the scale for the decimal and numeric columns, representing the number of digits after the decimal point.

#### References

```ruby
# can also use remove_reference
add_reference :users, :role [index: (true | false)] [polymorphic (true | false)]
```

#### Change method

[Parameters it supports](https://guides.rubyonrails.org/active_record_migrations.html#column-modifiers)

#### Reversible

Needs a reread

### Running Migrations

```sh
# Go to version x
rails db:migrate VERSION=20080906120000
```

- say, say with time, and suppress messages

### Schema dumps

load it with `rails db:schema:load`

- Types: ruby, sql

to resolve merge conflicts run `bin/rails db:migrate` to regenerate the schema file.

### Seeds

Quickly put some data in the database. fill up the `db/seeds.rb` and run `rails db:seed`

## AR Validations

Active Record uses the new_record? instance method to determine whether an object is already in the database or not

The following methods trigger validations, and will save the object to the database only if the object is valid:

- create
- create!
- save
- save!
- update
- update!
The bang versions (e.g. save!) raise an exception if the record is invalid. The non-bang versions don't: save and update return false, and create returns the object.

### :message

### :on

### Conditional validation 

```ruby
# paid_with_card is a defined method
validates :card_number, presence: true, if: :paid_with_card?

# Can use proc or lambda
validates :password, confirmation: true,
  unless: Proc.new { |a| a.password.blank? }

validates :password, confirmation: true, unless: -> { password.blank? }

# With options to group validations
with_options if: :is_admin? do |admin|
  admin.validates :password, length: { minimum: 10 }
  admin.validates :email, presence: true
end

  validates :mouse, presence: true,
      if: [Proc.new { |c| c.market.retail? }, :desktop?],
      unless: Proc.new { |c| c.trackpad.present? }
```

### Custom validations

## Associations

An author has many books.

```ruby
class Author < ApplicationRecord
  has_many :books, dependent: :destroy
end

class Book < ApplicationRecord
  belongs_to :author
end

# Creating a book belonging to a partiuclar author
@book = @author.books.create(published_at: Time.now)

# destroying an author deletes all its books as well
@author.destroy
```

### The types of associations

Rails supports six types of associations:

- `belongs_to`
- `has_one`

  The has_one relationship says that one of something is yours - that is, that something points back to you. For example, it makes more sense to say that a supplier owns an account than that an account owns a supplier.

- `has_many`
- `has_many :through` 
  
  the declaring model can be matched with zero or more instances of another model by proceeding through a third model.
  
  ```ruby
  class Physician < ApplicationRecord
    has_many :appointments
    has_many :patients, through: :appointments
  end

  class Appointment < ApplicationRecord
    belongs_to :physician
    belongs_to :patient
  end

  class Patient < ApplicationRecord
    has_many :appointments
    has_many :physicians, through: :appointments
  end
  ```

  The collection of join models can be managed via the has_many association methods. For example, if you assign:

  `physician.patients = patients`

  Then new join models are automatically created for the newly associated objects. If some that existed previously are now missing, then their join rows are automatically deleted.

  Automatic deletion of join models is direct, no destroy callbacks are triggered.

- `has_one :through`
- `has_and_belongs_to_many`

  ```ruby
  class Assembly < ApplicationRecord
    has_and_belongs_to_many :parts
  end

  class Part < ApplicationRecord
    has_and_belongs_to_many :assemblies
  end
  ```

-----------------

> Questions

-------------------

> This relation can be bi-directional when used in combination with belongs_to on the other model.

in a rails bi-directional relation, well the bi-directionality be reflected
in the database schema?

-----------------

You are building the profile tab for a new user on your site. You are already storing your user’s username and email, but now you want to collect demographic information like city, state, country, age and gender. Think – how many profiles should a user have? How would you relate this to the User model?

-----------------

Creating a post by supplying user id manually `p = Post.new(user_id: 1, link: "somthing.com")` does not display it when I query `u.posts`