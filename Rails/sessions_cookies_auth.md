# Sessions Cookies, And Authentication

## Cookies

- Cookies are key-value data pairs that are stored in the user’s browser until they reach their specified expiration date.
- You shouldn’t store anything in regular browser cookies that needs to either be secure or persisted across browser sessions.
  - It’s too easy for users to clear their cache and/or steal/manipulate unsecured cookies.
- Rails gives you access to a special hash called `cookies`, where each key-value pair is stored as a separate cookie on the user’s browser.
  - If you were to save `cookies[:hair-color] = "blonde"`, you’d be able to pull up your browser’s developer tools and see a cookie on the user’s browser that has a key of `hair-color` and a value of blonde. 
  - Delete it using `cookies.delete(:hair-color)`.
  - You can set their expiration dates

Rails also provides a signed cookie jar and an encrypted cookie jar for storing sensitive data. The signed cookie jar appends a cryptographic signature on the cookie values to protect their integrity. The encrypted cookie jar encrypts the values in addition to signing them, so that they cannot be read by the end-user.

## Sessions

- **Sessions** are the idea that your user’s state is somehow preserved when they click from one page to the next.
  - represents all the stuff your user does while you’ve chosen to "remember" them
- To identify a user’s session information, Rails stores a special secure and tamper-proof cookie on the user’s browser that contains their entire session hash
  - Whenever the user makes a request to your application, that request will also automatically include that session cookie (along with the other cookies) and you can use it to keep track of her logged-in state.

- The session is only avaliable in the controller and the view and can use one of several of different storage mechanisms:
  - `ActionDispatch::Session::CookieStore` - Stores everything on the client.
  - `ActionDispatch::Session::CacheStore` - Stores the data in the Rails cache.
  - `ActionDispatch::Session::ActiveRecordStore` - Stores the data in a database using Active Record (requires the activerecord-session_store gem).
  - `ActionDispatch::Session::MemCacheStore` - Stores the data in a memcached cluster (this is a legacy implementation; consider using CacheStore instead).

### Session database storage drawbacks

- With some database stores, your sessions won’t get cleaned up automatically.

  So you’ll have to go through and clean expired sessions on your own.

- You have to know how your database will behave when it’s full of session data.

  Are you using Redis as your session store? Will it try to keep all your session data in memory? Does your server have enough memory for that, or will it start swapping so badly you won’t be able to ssh in to fix it?

- You have to be more careful about when you create session data, or you’ll fill your database with useless sessions.

  For example, if you accidentally touch the session on every request, googlebot could create hundreds of thousands of useless sessions. And that would be a bad time.

### Sessions Hash

Rails gives you access to the `session` hash in an almost identical way to the above-mentioned `cookies` hash. Use the `session` variable in your views or controllers like so:

```rb
# app/controllers/users_controller.rb
...
# Set a session value
session[:current_user_id] = user.id

# Access a session value
some_other_variable_value = session[:other_variable_key]

# Reset a session key
session[:key_to_be_reset] = nil

# Reset the entire session
reset_session
...
```

`session` is an entire hash that gets put in the secure session cookie that expires when the user closes the browser. If you look in your developer tools, the “expiration” of that cookie is “session”. Each value in the cookies hash gets stored as an individual cookie.

So cookies and sessions are sort of like temporary free database tables for you to use that are unique to a given user and will last until you either manually delete them, they have reached their expiration date, or the session is ended (depending on what you specified).

- `session` and `cookies` aren’t really hashes, Rails just pretends they are so it’s easy for you to work with them. You can still consider them as hashes just because they act very similarly to hashes.
- You are size-limited in terms of how much you can store inside a session hash or browser cookie (~4kb). It is sufficient for any “normal” usage, but don’t go pretending either of these are actually substitutes for a database.

- Sessions are lazily loaded. If you don't access sessions in your action's code, they will not be loaded. Hence, you will never need to disable sessions, just not accessing them will do the job.

### Session best Practices

- Prepare for the session to go away at any time, program defensively!
- Don't store complex objects, prefer storing **references** to objects, not objecs themselves.
- Be deliberate! Use sessions with intent. Use them only when they make sense.

### How to debug session problems

1. Isonlate problem area as quickly as possible
2. Best tools for debugging session issues are about showing what user is sending and what user is receiving. (mitm proxy)
3. Domain settings on cookies may be wrong
4. Try decrypting cookies (CookieDecryptor gem)


## Flahses

- flash is a special hash (okay, a method that acts like a hash) that persists only from one request to the next.
  - The flash is a special part of the session which is cleared with each request.
- You can think of it as a `session` hash that self destructs after it’s opened.
- store `flash[:success]` (or whatever you’d like it called) and it will be available to your view on the next new request. 
- `flash.now[:error] = "Fix your submission!"` will be available immediately.
- As soon as the view accesses the hash, Rails erases the data.

- You can assign `:notice`, `:alert` or the general-purpose `:flash`:

```rb
redirect_to root_url, notice: "You have successfully logged out."
redirect_to root_url, alert: "You're stuck here!"
redirect_to root_url, flash: { referral_code: 1234 }
```

- `flash.keep`: persist flash value for another request
  - You can also use a key to keep only some kind of value `flash.keep(:notice)`

## Controller Filters

- The idea of filters is to run some code in your controller at very specific times.
  - For instance before any other code has been run.

- e.g. filter out unauthorized request:

  ```rb
  # app/controllers/users_controller
  # Will run before any action
  before_action :require_login
  
  # Will run only before specific actions
  # also supports except:
  before_action :require_login, only: [:edit, :update]

  ...
  private

  def require_login
    # do stuff to check if user is logged in
  end
  ```

- Filters are inherited so if you’d like a filter to apply to absolutely every controller action, put it in your `app/controllers/application_controller.rb` file.

- `after_action` filters have access to the response data.

- `around_action` filters are responsible for running their associated actions by yielding.

- You can implement filters as classes

## Authentication

- **Authentication**: Make sure that the user is who they say they are.
- **Authorization**: Yes, you may be signed in, but are you actually authorized to access what you’re trying to access?

Rails supports three types of authentication:
  - http_basic_authenticate_with  
  - authenticate_or_request_with_http_digest 
  - authenticate_or_request_with_http_token 

### Principles of Authentication

- Don’t store passwords in plain text in the database.
  - Databases often get leaked! Instead, you’ll store an encrypted "password digest" version of the password.
- Rails has a method called `#has_secure_password` which you just drop into your User model and it will add a lot of the functionality 
  - set up your User model to handle accepting `password` and `password_confirmation` attributes but you won’t actually persist those to the database. `has_secure_password` intercepts those values and converts them into the password digest for you.
- If your user wants to be “remembered”, you need a way to remember them for longer than just the length of the browser session.
  - create another column in your Users table for an encrypted `remember_token` (or whatever you’d like to call it). You’ll use that to store a random string for that user that will be used in the future to identify him/her.
  - You will drop the unencrypted token as a permanent cookie `using cookies.permanent[:remember_token]` into the user’s browser. 
  - That cookie will be submitted with each new request, so you can check with the encrypted version in the database to see who that user is whenever they make a request.

### Authentication flow 

A generic step-by-step overview:

1. Add a column to your Users table to contain the user’s `password_digest`.
1. When the user signs up, turn the password they submitted into digest form and then store THAT in the new database column by adding the `has_secure_password` method to your User model.
1. Don’t forget any necessary validations for password and password confirmation length.
1. Build a sessions controller (and corresponding routes) and use the `#authenticate` method to sign in the user when the user has submitted the proper credentials using the signin form.
1. Allow the user to be remembered by creating a `remember_token` column in the Users table and saving that token as a permanent cookie in the user’s browser. Reset on each new signin.
1. On each page load that requires authentication (and using a #before_action in the appropriate controller(s)), first check the user’s cookie remember_token against the database to see if he’s already signed in. If not, redirect to the signin page.
1. Make helper methods as necessary to let you do things like easily determine if a user is signed in or compare another user to the currently signed in user.
1. Profit.

## Devise

- Gem that handles all authentication stuff
- Made up of 10 modules (and you can choose which ones you want to use).
  - Need to run a database migration to add their additional fields to your Users table.


## Code snippits

### Find current user

```rb
class ApplicationController < ActionController::Base

  private

  # Finds the User with the ID stored in the session with the key
  # :current_user_id This is a common way to handle user login in
  # a Rails application; logging in sets the session value and
  # logging out removes it.
  def current_user
    @_current_user ||= session[:current_user_id] &&
      User.find_by(id: session[:current_user_id])
  end
end

class LoginsController < ApplicationController
  # "Create" a login, aka "log the user in"
  def create
    if user = User.authenticate(params[:username], params[:password])
      # Save the user ID in the session so it can be used in
      # subsequent requests
      session[:current_user_id] = user.id
      redirect_to root_url
    end
  end
end
```

### Remove from session 

```rb
class LoginsController < ApplicationController
  # "Delete" a login, aka "log the user out"
  def destroy
    # Remove the user id from the session
    session.delete(:current_user_id)
    # Clear the memoized current user
    @_current_user = nil
    # Display message to use on next request
    flash[:notice] = "You have successfully logged out."
    redirect_to root_url
  end
end
```

- To reset the entire session, use `reset_session`.

### Display flash

```erb
# app/views/layouts/application.html.erb
...
<% flash.each do |name, message| %>
  <div class="<%= name %>"><%= message %></div>
<% end %>
```

### Requiring user log in to perform actions

```rb
class ApplicationController < ActionController::Base
  before_action :require_login

  private

  def require_login
    unless logged_in?
      flash[:error] = "You must be logged in to access this section"
      redirect_to new_login_url # halts request cycle
    end
  end
end
```

...but actually give them the ability to log in, prevent filter running before certain actions

```rb
class LoginsController < ApplicationController
  skip_before_action :require_login, only: [:new, :create]
end
```a