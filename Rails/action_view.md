# Action View

- use an instance variable from your view, just call it the same way you would in the controller: `@user.first_name` 

- Layouts live in the directory `app/views/layouts`. It’s got the basic tags you need in all webpages (e.g. `<html>` and `<body>`) and a couple snippets of code that load up the JavaScript and CSS files your webpage will need. 

  You’ll want to put anything that’s needed across all your webpages into the layout. Usually this is stuff like navbars and footers and snippets of code for displaying flash messages.

- `<%=` displays whatever is returned inside the ERB tags.
- `<%` will execute the code but will not actually display anyth
- `<%#` is used to comment and will not execute.

- To suppress leading and trailing whitespaces, you can use <%- -%> interchangeably with <% and %>.

## View Partials

Render partial _user_form.html.erb

```erb
  <!-- app/views/users/new.html.erb -->
  <div class="new-user-form">
    <%= render "user_form" %>
  </div>
```

- to share partials across multiple view templates that are in multiple controllers, save them in their own folder called `app/views/shared` render them using `<%= render "shared/some_partial"%>`

## Passing Local Variables to Partials

- Don't depend on view variables in partials, as the calling view may not have that variable.

- `render` is just a regular method and it lets you pass it an `options` hash. One of those options is the `:locals` key, which will contain the variables you want to pass.

```erb
<!-- Pass variable @user to partial -->
  <%= render partial: "shared/your_partial", :locals => { :user => user } %>

  <!--Same thing-->
  <%= render "shared/your_partial", :user => user %>

  <!--Same thing (other example) -->
  <%= render "product", product: @product %>

```

By default `ActionView::Partials::PartialRenderer` has its object in a local variable with the same name as the template. So, given:

```erb
<%= render partial: "product" %>
```

within `_product` partial we'll get `@product` in the local variable `product`, as if we had written:

```erb
<%= render partial: "product", locals: { product: @product } %>
```

### If we want to pass a different variable

```erb
<%= render partial: "product", object: @item %>
```

### Specify different name for local variable

```erb
<%= render partial: "product", object: @item, as: "item" %>

<%= render partial: "product", locals: { item: @item } %>

```

### Rendering a collection

```erb
<%= render partial: "product", collection: @products %>

<%= render @products %>
```

### Spacer element

specify a second partial to be rendered between instances of the main partial by using the :spacer_template option:

```erb
<%= render partial: @products, spacer_template: "product_ruler" %>
```

## Partial layouts

inside show.html.erb

```erb
<%= render partial: 'article', layout: 'box', locals: { article: @article } %>
```

The `box` layout simply wraps the `_article` partial in a div:

```erb
<div class='box'>
  <%= yield %>
</div>
```

Or alternatively, inside show.html.erb:

```erb
<% render(layout: 'box', locals: { article: @article }) do %>
  <div>
    <p><%= article.body %></p>
  </div>
<% end %>


```

## View Paths

- When rendering a response, the controller needs to resolve where the different views are located. By default, it only looks inside the app/views directory.

- add other locations and give them certain precedence when resolving paths using the prepend_view_path and append_view_path methods.

## Localized views

suppose you have an `ArticlesController` with a `show` action. By default, calling this action will render `app/views/articles/show.html.erb`. But if you set `I18n.locale = :de`, then `app/views/articles/show.de.html.erb` will be rendered instead.

setting `I18n.locale = :de` and creating `public/500.de.html` and `public/404.de.html` would allow you to have localized rescue pages.

