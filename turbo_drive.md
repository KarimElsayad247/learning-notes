# Turbo Drive

Handles requests to update page without full reload 🚀🚀🚀

use a button or form for anything that isn’t a GET request

You have Two types of visits: application and restoration

- Application visit, a visit with a Drive action of advance or replace.
- Restoration visit, a visit with a Drive action of restore.

- Restoration visits are visits with the action of restore. This is used by Turbo Drive internally and you should not annotate a link with an action of restore.

- By default, link clicks are sent with `GET` requests. However, Turbo Drive will scan `<a>` tags in your application for the `turbo-method` attribute to override the `GET` action.

- You can disable Turbo Drive by adding data-turbo="false" directly on your links or on the parent containing them.

```erb
<div data-turbo="false">
  <%= link_to "foo", "bar" %>
  <%= link_to "baz", "qux", data: { turbo: "true" } %>
</div>
```

- To specify that following a link should trigger a replace visit, annotate the link with data-turbo-action="replace":

```html
<a href="/edit" data-turbo-action="replace">Edit</a>
```

- Turbo Streams introduces a `<turbo-stream>` element with seven basic actions: append, prepend, replace, update, remove, before, and after. 

## Forms

- For a form response, Turbo expects the server to return an HTTP status of HTTP 303 or, in other words, a redirect.


