# Asset pipeline

Bundles all assets by putting all files of a certain type in one big file and minifies it. 

## In erb files:

```erb
<%= stylesheet_link_tag "application", media: "all" %>
<%= javascript_include_tag "application" %>
```

## Manifest files

### Javascript

The lines starting with //= tell Rails which files to go find and include. 

jQuery also isn’t now included out of the box. Rails now uses the rails_ujs instead

### CSS

#### Namespacing

The HTML

```html
<div class="user">
  <div class="container">
    <!-- a bunch of code for displaying the user -->
  </div>
</div>
```

The CSS

```css
.user .container{
  // style stuff
}

```

### Unescaping HTML

 If you just write something like this is the \<strong>BODY\</strong> of my post and then try to display it in a view later, the \<strong> tags will just be regular text… they will literally say ‘\<strong>’. That’s called “escaping” the characters.

To get your views to actually render HTML as HTML, you need to let Rails know that the code is safe to run. Otherwise, it’s easy for a malicious attacker to inject code like \<script> tags that cause major issues when you try to render them.

To tell Rails a string is safe, just use the method raw in your view template, for example:

```erb
  <%= raw "<p>hello world!</p>" %>   <!-- this will create real <p> tags -->
```

## How to Use the Asset Pipeline

- Assets can still be placed in the public hierarchy. Any assets under public will be served as static files by the application or web server when config.public_file_server.enabled is set to true. 

- You should use app/assets for files that must undergo some pre-processing before they are served.

- `app/assets` is for assets that are owned by the application, such as custom images, JavaScript files, or stylesheets.

- `lib/assets` is for your own libraries' code that doesn't really fit into the scope of the application or those libraries which are shared across applications.

- `vendor/assets` is for assets that are owned by outside entities, such as code for JavaScript plugins and CSS frameworks. Keep in mind that third party code with references to other files also processed by the asset Pipeline (images, stylesheets, etc.), will need to be rewritten to use helpers like asset_path.

### The index file

Sprockets uses files named index (with the relevant extensions) for a special purpose.

For example, if you have a jQuery library with many modules, which is stored in `lib/assets/javascripts/library_name`, the file `lib/assets/javascripts/library_name/index.js` serves as the manifest for all files in this library. This file could include a list of all the required files in order, or a simple `require_tree` directive.

### CSS

- data URI - a method of embedding the image data directly into the CSS file - you can use the `asset_data_uri` helper. Note that the closing tag cannot be of the style `-%>`.

```erb
#logo { background: url(<%= asset_data_uri 'logo.png' %>) }
```

sass-rails provides -url and -path helpers (hyphenated in Sass, underscored in Ruby) for the following asset classes: image, font, video, audio, JavaScript and stylesheet.

`image-url("rails.png")` returns `url(/assets/rails.png)`
`image-path("rails.png")` returns `"/assets/rails.png"`

The more generic form can also be used:

`asset-url("rails.png")` returns `url(/assets/rails.png)`
`asset-path("rails.png")` returns `"/assets/rails.png"`

- If you want to use multiple Sass files, you should generally use the Sass @import rule instead of these Sprockets directives. When using Sprockets directives, Sass files exist within their own scope, making variables or mixins only available within the document they were defined in.

- In development mode, or if the asset pipeline is disabled, when a sass file is requested it is processed by the processor provided by the `sass-rails` gem and then sent back to the browser as CSS. When asset pipelining is enabled, this file is preprocessed and placed in the `public/assets` directory for serving by either the Rails app or web server.