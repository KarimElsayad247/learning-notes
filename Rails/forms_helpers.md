# Forms helpers

- Rails by default automatically protects you from cross-site request forgery. it generates an "authenticity token"

- method `form_authenticity_token` generates the token

- By defalut forms are submitted by Turbo Drive.

- If you find yourself submitting a form and nothing is happening, chances are you forgot to return the correct status. You can confirm in the server logs when you submit the form.

## form_with helper

Pass it arguments to tell it which path to submit to (the default is the current page) and which method to use.

```erb
<%= form_with(url: "/search", method: "get") do %>
  <%= label_tag(:query, "Search for:") %>
  <%= text_field_tag(:query) %>
  <%= submit_tag("Search") %>
<% end %>
```

Creates 

```html
<form accept-charset="UTF-8" action="/search" method="get">
  <label for="query">Search for:</label>
  <input id="query" name="query" type="text" />
  <input name="commit" type="submit" value="Search" data-disable-with="Search" />
</form>
```

The best part about form_with is that if you just pass it a model object like `@article`, Rails will check for you if the object has been saved yet. If it’s a new object, it will send the form to your `#create` action. If the object has been saved before, so we know that we’re editing an existing object, it will send the object to your `#update` action instead. This is done by automatically generating the correct URL when the form is created.

Using a model infers both the URL and scope

```erb
<%= form_with model: @article do |form| %>
  <%= form.text_field :title %>
  <%= form.submit "Create" %>
<% end %>
```

### Error handling 

Forms helpers handle errors automatically too! If a form is rendered for a specific model object, like using `form_with model: @article`, Rails will check for errors and, if it finds any, it will automatically wrap a special `<div>` element around that field with the class `field_with_errors` so you can write whatever CSS you want to make it stand out. 

### Patch and delete

most browsers don't support methods other than "GET" and "POST" when it comes to submitting forms.

Rails works around this issue by emulating other methods over POST with a hidden input named "_method", which is set to reflect the desired method:

`form_with(url: search_path, method: "patch")`

When rendering a form, submission buttons can override the declared method attribute through the formmethod: keyword:

```erb
<%= form_with url: "/posts/1", method: :patch do |form| %>
  <%= form.button "Delete", formmethod: :delete, data: { confirm: "Are you sure?" } %>
  <%= form.button "Update" %>
<% end %>
```

### Name spacing

If you have created namespaced routes, form_with has a nifty shorthand for that too. If your application has an admin namespace then

```rb
form_with model: [:admin, @article]
```

will create a form that submits to the ArticlesController inside the admin namespace (submitting to admin_article_path(@article) in the case of an update)