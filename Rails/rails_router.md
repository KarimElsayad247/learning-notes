# [Rails router](https://guides.rubyonrails.org/routing.html)


`GET /patients/17` 

matches

`get '/patients/:id', to: 'patients#show'`

the request is dispatched to the patients controller's show action with { id: '17' } in params.

You can also generate paths and URLs. If the route above is modified to be:

```ruby
get '/patients/:id', to: 'patients#show', as: 'patient'
```

and your application contains this code in the controller:

```ruby
@patient = Patient.find(params[:id])
```

and this in the corresponding view:

```ruby
<%= link_to 'Patient Record', patient_path(@patient) %>
```

then the router will generate the path /patients/17

## Rails defaults

a single call to `resources :photos` will generate all 7 CURD routes index, show, new, edit, create, update, and destroy. This is called a resourceful route.

### Path and helpers

`photos_path` returns `/photos`
new_photo_path returns `/photos/new`
`edit_photo_path(:id)` returns `/photos/:id/edit` (for instance, `edit_photo_path(10)` returns `/photos/10/edit)`
`photo_path(:id)` returns `/photos/:id` (for instance, `photo_path(10)` returns `/photos/10)`

- Each of these helpers has a corresponding _url helper (such as photos_url) which returns the same path prefixed with the current host, port, and path prefix.

### Singular resource

```ruby
get 'profile', to: 'users#show'
```

### [Controller Namespaces and Routing](https://guides.rubyonrails.org/routing.html#controller-namespaces-and-routing)

```ruby
namespace :admin do
  resources :articles, :comments
end
```

Limit nesting to only one level deep http://weblog.jamisbuck.org/2007/2/5/nesting-resources

#### Shallow nesting

This

```ruby
resources :articles do
  resources :comments, shallow: true
end
```

is equsl to this

```ruby
resources :articles do
  resources :comments, only: [:index, :new, :create]
end
resources :comments, only: [:show, :edit, :update, :destroy]
```

## Routing concerns

Routing concerns allow you to declare common routes that can be reused inside other resources and routes.

Define:

```ruby
concern :commentable do
  resources :comments
end

concern :image_attachable do
  resources :images, only: :index
end
```

Use:

```ruby
resources :messages, concerns: :commentable

resources :articles, concerns: [:commentable, :image_attachable]
```

## More routing helpers

with this

```ruby
resources :magazines do
  resources :ads
end
``` 

use this

```ruby
<%= link_to 'Ad details', magazine_ad_path(@magazine, @ad) %>

<%= link_to 'Ad details', url_for([@magazine, @ad]) %>

<%= link_to 'Ad details', [@magazine, @ad] %>

# And for other actions
<%= link_to 'Edit Ad', [:edit, @magazine, @ad] %>

```

## Non Resourceful routes

```ruby
# :id is an optional parameter, denoted by parentheses.
get 'photos(/:id)', to: 'photos#display'

get 'photos/:id/:user_id', to: 'photos#show'

# with_user is a static segment
get 'photos/:id/with_user/:user_id', to: 'photos#show'

# Defining defaults
get 'photos/:id', to: 'photos#show', defaults: { format: 'jpg' }

# You can also use a defaults block to define 
# the defaults for multiple items:
defaults format: :json do
  resources :photos
end

# Naming a route. This will create logout_path and
# logout_url route helpers
get 'exit', to: 'sessions#destroy', as: :logout

```

You can also use this to override routing methods defined by resources by placing custom routes before the resource is defined, like this:

```ruby
get ':username', to: 'users#show', as: :user
resources :users
```

This will define a user_path method that will be available in controllers, helpers, and views that will go to a route such as `/bob`. 



### Segment constraints

```ruby
#  match paths such as /photos/A12345, but not /photos/893
get 'photos/:id', to: 'photos#show', constraints: { id: /[A-Z]\d{5}/ }

# same thing
get 'photos/:id', to: 'photos#show', id: /[A-Z]\d{5}/
```

## Customizing Resourceful Routes

- A `namespace` is a type of scope with `:as`, `:module` and `:path` applied.

```rb
namespace "admin" do
  resources :contexts
end
```

is the same as

```rb
scope "/admin", as: "admin", module: "admin" do
  resources :contexts
end
```


```rb
# use custom controller
resources :photos, controller: 'images'

# Namespaced controllers
resources :user_permissions, controller: 'admin/user_permissions'

# Constraints
resources :photos, constraints: { id: /[A-Z][A-Z][0-9]+/ }

# You can specify a single constraint to apply to 
# a number of routes by using the block form
constraints(id: /[A-Z][A-Z][0-9]+/) do  ///// # this regex breaks syntax highlighting in vscode 
  resources :photos
  resources :accounts
end

# This will only override helper names, but not controller
resources :photos, as: 'images'

# Override only new and edit segments, this changes it in
# the url, but not controller.
resources :photos, path_names: { new: 'make', edit: 'change' }

# Doing it for all routes
scope path_names: { new: 'make' } do
  # rest of your routes
end

# prevent name collisions between routes using a path scope.
scope 'admin' do
  resources :photos, as: 'admin_photos'
end

resources :photos

# And essentially same thing.
# This will generate routes such as admin_photos_path 
# and admin_accounts_path which map to /admin/photos and 
# /admin/accounts respectively.
scope 'admin', as: 'admin' do
  resources :photos, :accounts
end

resources :photos, :accounts

# Restricting created routes

# Only create these
resources :photos, only: [:index, :show]

# Create all but these
resources :photos, except: :destroy

```

## Route globbing and wildcard matching

This route would match `photos/12` or `/photos/long/path/to/12`, setting `params[:other]` to "12" or "long/path/to/12". The segments prefixed with a star are called "wildcard segments".

```rb
get 'photos/*other', to: 'photos#unknown'
```

would match `books/some/section/last-words-a-memoir` with `params[:section]` equals `'some/section'`, and `params[:title]` equals `'last-words-a-memoir'`.

```rb
get 'books/*section/:title', to: 'books#show'
```

## Redirection

```rb
# redirect any path to another path
# returns response 301 moved permanently
get '/stories', to: redirect('/articles')

# You can also reuse dynamic segments
get '/stories/:name', to: redirect('/articles/%{name}')

# use the :status option to change the response status:
get '/stories/:name', to: redirect('/articles/%{name}', status: 302)
```