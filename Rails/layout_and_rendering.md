# Layout and ordering

From Controller POV, 3 ways to create http response:

- Call `render` to create a full response to send back to the browser
- Call `redirect_to` to send an HTTP redirect status code to the browser
- Call `head` to create a response consisting solely of HTTP headers to send back to the browser

- if you do not explicitly render something at the end of a controller action, Rails will automatically look for the `action_name.html.erb` template in the controller's view path and render it. 

## Using render

You can render the default view for a Rails template, or a specific template, or a file, or inline code, or nothing at all. You can render text, JSON, or XML. You can specify the content type or HTTP status of the rendered response as well.

- use render with the name of the view to use different template 

  ```rb
  render "edit"

  render :edit, status: :unprocessable_entity

  # also all same thing
  render :edit
  render action: :edit
  render action: "edit"
  render "books/edit"
  render template: "books/edit"

  # Rendering a template from another controller.
  # render accepts the full path (relative to app/views) 
  # of the template to render.
  render "products/show"

  # Same thing but explicit.
  # The :template option was required on Rails 2.2 and earlier
  render template: "products/show"
  ```


- send plain text back to the browser by using the :plain option to render. most useful when you're responding to Ajax or web service requests that are expecting something other than proper HTML.

  By default, if you use the :plain option, the text is rendered without using the current layout. If you want Rails to put the text into the current layout, you need to add the layout: true option and use the .text.erb extension for the layout file.

  ```rb
  render plain: "OK"
  ```

- There is also:

```rb
# send an HTML string back to the browser .
# HTML entities will be escaped if the string is not 
# composed with html_safe-aware APIs.
render html: helpers.tag.strong('Not Found')

# send JSON.
# You don't need to call to_json on the object that you 
# want to render.
render json: @product

# send XML
# same as above, no need for to_xml
render xml: @product

# Render vanilla javascript
render js: "alert('Hello Rails');"

# Raw content. Using :plain or :html might be appropriate
# most of the time.
# your response returned from this render option will be
# text/plain, as that is the default content type of 
# Action Dispatch response.
render body: "raw"

# Raw file from absolute path.
# Using this in combination with user input may lead to 
# security problems.
# send_file is often a faster and better option if a layout
# isn't required.
render file: "#{Rails.root}/public/404.html", layout: false

# Rails can render objects responding to :render_in.
render MyRenderable.new
```

### Options for render

Calls to the `render method generally accept six options:

- `:content_type`: change MIME type in header
- `:layout`: other layout or no layout at all
- `:location`: set HTTP location header
- `:status`: HTTP status code to return. [All optoins here](https://guides.rubyonrails.org/layouts_and_rendering.html#the-status-option).

  If you try to render content along with a non-content status code (100-199, 204, 205, or 304), it will be dropped from the response.

- `:formats`: Rails uses the format specified in the request (or `:html` by default). You need to create a template for the format.

- `:variants`:  look for template variations of the same format. e.g. `render variants: [:mobile, :desktop]`

## Layouts

### Finding layouts

1. Rails first looks for a file in `app/views/layouts` with the same base name as the controller. For example, rendering actions from the `PhotosController` class will use `app/views/layouts/photos.html.erb`

2. If there is no such controller-specific layout, Rails will use `app/views/layouts/application.html.erb`

3. If there is no .erb layout, Rails will use a .builder layout if one exists.

### More layout stuff

- Specify layout for controller. all of the views rendered by the `ProductsController` will use `app/views/layouts/inventory.html.erb` as their layout.

  ```rb
  class ProductsController < ApplicationController
    layout "inventory"
    #...
  end
  ```

- Specify layout for the entire application. all of the views in the entire application will use `app/views/layouts/main.html.erb` for their layout.

  ```rb
  class ApplicationController < ActionController::Base
    layout "main"
    #...
  end
  ```

- Choose layout at runtime. 

  ```rb
  class ProductsController < ApplicationController
    layout :products_layout

    def show
      @product = Product.find(params[:id])
    end

    private
      def products_layout
        @current_user.special? ? "special" : "products"
      end
  end

  # Can also use a proc
  class ProductsController < ApplicationController
    layout Proc.new { |controller| controller.request.xhr? ? "popup" : "application" }
  end

  # Conditional layouts. They support :only and :except
  class ProductsController < ApplicationController
    layout "product", except: [:index, :rss]
  end
  ```

- Layout declarations cascade downward in the hierarchy, and more specific layout declarations always override more general ones.

- Similar to the Layout Inheritance logic, if a template or partial is not found in the conventional path, the controller will look for a template or partial to render in its inheritance chain. This makes `app/views/application/` a great place for your shared partials. [More details here.](https://guides.rubyonrails.org/layouts_and_rendering.html#template-inheritance)

- `render` does not return, so you might run into a double render error. 

- Implicit render done by ActionController detects if render has been called, so this is ok

  ```rb
  def show
    @book = Book.find(params[:id])
    if @book.special?
      render action: "special_show"
    end
  end
  ```

## Using redirect_to

It tells the browser to send a new request for a different URL. For example, you could redirect from wherever you are in your code to the index of photos in your application with `redirect_to photos_url`.

- use `redirect_back` to return the user to the page they just came from. This location is pulled from the `HTTP_REFERER` header which is not guaranteed to be set by the browser, so you must provide the `fallback_location` to use in this case.

```rb
redirect_back(fallback_location: root_path)
```

- `redirect_to` and `redirect_back` do not halt and return immediately from method execution, but simply set HTTP responses. Statements occurring after them in a method will be executed. You can halt by an explicit return or some other halting mechanism, if needed.

- Rails uses HTTP status code 302, a temporary redirect, by default when you call `redirect_to`. to use another one, use `:status` option just like with `render`.

```rb
redirect_to photos_path, status: 301
```

## Structuring layouts

 Within a layout, you have access to three tools for combining different bits of output to form the overall response:

- Asset tags
- `yield` and `content_for`
- Partials

### Asset tags 

Asset tag helpers provide methods for generating HTML that link views to feeds, JavaScript, stylesheets, images, videos, and audios. There are six asset tag helpers available in Rails:

- `auto_discovery_link_tag`
- `javascript_include_tag`
- `stylesheet_link_tag`
- `image_tag`
- `video_tag`
- `audio_tag`

You can use these tags in layouts or other views, although the auto_discovery_link_tag, javascript_include_tag, and stylesheet_link_tag, are most commonly used in the <head> section of a layout.

- **Important:** The asset tag helpers do not verify the existence of the assets at the specified locations; they simply assume that you know what you're doing and generate the link.

### Javascript 

To link to a JavaScript file that is inside a directory called javascripts inside of one of `app/assets`, `lib/assets` or `vendor/assets`:

```erb
<%= javascript_include_tag "main" %>

<!-- To include multiple files if they are in same folder: -->
<%= javascript_include_tag "main", "columns" %>

<!-- Or if they are in different folders -->
<%= javascript_include_tag "main", "/photos/columns" %>

<!-- or a link -->
<%= javascript_include_tag "http://example.com/main.js" %>
```

### CSS

Generally follows the same rules as Javascript include tags.

By default, the `stylesheet_link_tag` creates links with `rel="stylesheet"`. You can override this default by specifying an appropriate option (`:rel`):

```erb
<%= stylesheet_link_tag "main_print", media: "print" %>
```

### Images

The `image_tag` helper builds an HTML `<img />` tag to the specified file. By default, files are loaded from `public/images`.

**Note that you must specify the extension of the image.**

[There are a lot more details](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-images-with-the-image-tag)

### Vides

Similar to images. [Just look at the source](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-videos-with-the-video-tag)

### Audio

[Look at the source here too](https://guides.rubyonrails.org/layouts_and_rendering.html#linking-to-audio-files-with-the-audio-tag)


## Yields

Within the context of a layout, `yield` identifies a section where content from the view should be inserted.

**This is a layout**
```erb
<html>
  <head>
  <%= yield :head %>
  </head>
  <body>
  <%= yield %>
  </body>
</html>
```

The main body of the view will always render into the unnamed `yield`. To render content into a named yield, you use the `content_for` method.

## Content for

Insert content into a named yield block in your layout.

**This is a view**
```erb
<% content_for :head do %>
  <title>A simple page</title>
<% end %>

<p>Hello, Rails!</p>
```

## Partials

- A partial can use its own layout. explicitly specifying `:partial` is required when passing additional options such as `:layout`

  ```erb
  <%= render partial: "link_area", layout: "graybar" %>
  ```

- You can also pass local variables into partials

  ```erb
  <!-- new.html.erb -->
  <h1>New zone</h1>
  <%= render partial: "form", locals: {zone: @zone} %>


  <!-- edit.html.erb -->
  <h1>Editing zone</h1>
  <%= render partial: "form", locals: {zone: @zone} %>


  <!-- _form.html.erb -->
  <%= form_with model: zone do |form| %>
    <p>
      <b>Zone name</b><br>
      <%= form.text_field :name %>
    </p>
    <p>
      <%= form.submit %>
    </p>
  <% end %>
  ```
  
  Although the same partial will be rendered into both views, Action View's submit helper will return "Create Zone" for the new action and "Update Zone" for the edit action.