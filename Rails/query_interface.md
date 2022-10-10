# Active record Query interface

## Methods

### Retrieving objects from the database

- `annotate`
- `find`
- `create_with`
- `distinct`
- `eager_load`
- `extending`
- `extract_associated`
- `from`
- `group`
- `having`
- `includes`
- `joins`
- `left_outer_joins`
- `limit`
- `lock`
- `none`
- `offset`
- `optimizer_hints`
- `order`
- `preload`
- `readonly`
- `references`
- `reorder`
- `reselect`
- `reverse_order`
- `select`
- `where`

### Sanitization 

- `sanitize_sql_like`
- Avoiding sql injection:

  ```rb
  # Prefer this
  Book.where("title = ?", params[:title])
  
  # to this, since putting parame inside string directly
  # won't allow rails to escape sql
  Book.where("title = #{params[:title]}")
  ```

### Hash condition

Only equality, range, and subset checking are possible with Hash conditions.

```rb
# Equality 
Book.where(out_of_print: true)
Book.where('out_of_print' => true)

# In the case of a belongs_to relationship, an association
# key can be used to specify the model if an Active Record
# object is used as the value.
author = Author.first
Book.where(author: author)
Author.joins(:books).where(books: { author: author })

# Range
Book.where(created_at: (Time.now.midnight - 1.day)..Time.now.midnight)

# Beginless and endless ranges are supported and can be
# used to build less/greater than conditions.
Book.where(created_at: (Time.now.midnight - 1.day)..)

# Subset Conditions
Customer.where(orders_count: [1,3,5])
```

### More where

```rb
# Not condition 
Customer.where.not(orders_count: [1,3,5])

# Not null
Customer.where.not(nullable_country: nil)

# OR condition
Customer.where(last_name: 'Smith').or(Customer.where(orders_count: [1,3,5]))

# AND condition: built by chaining where conditions.
Customer.where(last_name: 'Smith').where(orders_count: [1,3,5]))

## AND conditions for the logical intersection between
## relations
Customer.where(id: [1, 2]).and(Customer.where(id: [2, 3]))

# Ordering

## order a set of records in ascending order by 
## the created_at field
Book.order(:created_at)
Book.order("created_at")

## You can specift ASC or DESC
Book.order(created_at: :desc)
Book.order(created_at: :asc)
Book.order("created_at DESC")
Book.order("created_at ASC")

## Or order by multiple fields
Book.order(title: :asc, created_at: :desc)
Book.order(:title, created_at: :desc)
Book.order("title ASC, created_at DESC")
Book.order("title ASC", "created_at DESC")

## if you call order multiple times, subsequent orders 
## will be appended to the first
Book.order("title ASC").order("created_at DESC")
```

## Select

```rb
# select only isbn and out_of_print columns
Book.select(:isbn, :out_of_print)
Book.select("isbn, out_of_print")

# grab a single record per unique value in a certain field
Customer.select(:last_name).distinct

# Returns unique last_names
query = Customer.select(:last_name).distinct

# You can remove the uniqueness constraints.
# Returns all last_names, even if there are duplicates
query.distinct(false)
```

## Limit and offset

```rb
# return a maximum of 5 customers.
# because it specifies no offset it will return 
# the first 5 in the table.
Customer.limit(5)

# return instead a maximum of 5 customers beginning 
# with the 31st
Customer.limit(5).offset(30)
```

## Group and having

```rb
# find a collection of the dates on which 
# orders were created.
# this will give you a single Order object for each date
# where there are orders in the database.
Order.select("created_at").group("created_at")

# Get total of grouped items
Order.group(:status).count
=> {"being_packed"=>7, "shipped"=>12}

# This returns the date and total price for each order
# object, grouped by the day they were ordered and where
# the total is more than $200.
Order.select("created_at, sum(total) as total_price")
     .group("created_at")
     .having("sum(total) > ?", 200)
```


## Chekcing for existence

```rb
# it will return either true or false.
Customer.exists?(1)

# Can take multiple values, but Will return true
# if any of them exists 
Customer.exists?(id: [1,2,3])
Customer.exists?(first_name: ['Jane', 'Sergei'])

# Can be used without any arguments on a model
# or a relation.
# returns true if there is at least one customer with the
# first_name 'Ryan' and false otherwise.
Customer.where(first_name: 'Ryan').exists?

# returns false if the customers table is empty 
# and true otherwise.
Customer.exists?

# From the Guide:
# via a model
Post.any?
Post.many?

# via a relation
Post.where(published: true).any?
Post.where(published: true).many?

# via an association
Post.first.categories.any?
Post.first.categories.many?
```