# Action Controller

- A controller can thus be thought of as a middleman between models and views. It makes the model data available to the view, so it can display that data to the user, and it saves or updates user data to the model.

- A controller should be thin

## Controller Naming Convention

The naming convention of controllers in Rails favors pluralization of the last word in the controller's name, although it is not strictly required (e.g. ApplicationController). For example, ClientsController is preferable to ClientController, SiteAdminsController is preferable to SiteAdminController or SitesAdminsController, and so on.

This allows using default route generators

## Methods and Actions

- Rails will by default render the new.html.erb view unless the action says otherwise.

- By creating a new Client, the new method can make a @client instance variable accessible in the view:

## Parameters

- Rails does not make any distinction between query string parameters and POST parameters, and both are available in the `params` hash in your controller

- Params hash can contain nested arrays and hashes. 

  - To send an array of values, append an empty pair of square brackets "[]" to the key name:

    ```
    GET /clients?ids[]=1&ids[]=2&ids[]=3
    ```

    The value of params[:ids] will now be ["1", "2", "3"].

  - To send a hash, append key name inside brackets

    ```html
    <form accept-charset="UTF-8" action="/clients" method="post">
      <input type="text" name="client[name]" value="Acme" />
      <input type="text" name="client[phone]" value="12345" />
      <input type="text" name="client[address][postcode]" value="12345" />
      <input type="text" name="client[address][city]" value="Carrot City" />
    </form>
    ```
    When this form is submitted, the value of `params[:client]` will be `{ "name" => "Acme", "phone" => "12345", "address" => { "postcode" => "12345", "city" => "Carrot City" } }`. Note the nested hash in `params[:client][:address]`.

- The params object acts like a Hash, but lets you use symbols and strings interchangeably as keys.

- The params hash will always contain the `:controller` and `:action` keys, but you should use the methods `controller_name` and `action_name` instead

- [Routing Parameters](https://guides.rubyonrails.org/action_controller_overview.html#routing-parameters)

- [Default URL options](https://guides.rubyonrails.org/action_controller_overview.html#default-url-options)

### [Strong Parameters](https://guides.rubyonrails.org/action_controller_overview.html#strong-parameters)

Take a look at the page for list of permitted values

```ruby
    # Using a private method to encapsulate the permissible parameters
    # is just a good pattern since you'll be able to reuse the same
    # permit list between create and update. Also, you can specialize
    # this method with per-user checking of permissible attributes.
    def person_params
      params.require(:person).permit(:name, :age)
    end
```

- Extreme care should be taken when using permit!, as it will allow all current and future model attributes to be mass-assigned.

- You may want to also use the permitted attributes in your `new` action. This raises the problem that you can't use `require` on the root key because, normally, it does not exist when calling `new`:

```ruby
# using `fetch` you can supply a default and use
# the Strong Parameters API from there.
params.fetch(:blog, {}).permit(:title, :author)
```

#### Nested parameters

```ruby
params.permit(:name, { emails: [] },
              friends: [ :name,
                         { family: [ :name ], hobbies: [] }])
```

## Sessions

### The Flash

- The flash is a special part of the session which is cleared with each request. 

- The flash is accessed via the flash method, the flash is a hash

```ruby
class LoginsController < ApplicationController
  def destroy
    session.delete(:current_user_id)
    flash[:notice] = "You have successfully logged out."
    redirect_to root_url
  end
end
```

## Rendering Json and XML

```rb
class UsersController < ApplicationController
  def index
    @users = User.all
    respond_to do |format|
      format.html # index.html.erb
      format.xml  { render xml: @users }
      format.json { render json: @users }
    end
  end
end
```

## Streaming and File download

```rb
require "prawn"
class ClientsController < ApplicationController
  # Generates a PDF document with information on the client and
  # returns it. The user will get the PDF as a file download.
  def download_pdf
    client = Client.find(params[:id])
    send_data generate_pdf(client),
              filename: "#{client.name}.pdf",
              type: "application/pdf"
  end

  private
    def generate_pdf(client)
      Prawn::Document.new do
        text client.name, align: :center
        text "Address: #{client.address}"
        text "Email: #{client.email}"
      end.render
    end
end
```

```rb
class ClientsController < ApplicationController
  # Stream a file that has already been generated and stored on disk.
  def download_pdf
    client = Client.find(params[:id])
    send_file("#{Rails.root}/files/clients/#{client.id}.pdf",
              filename: "#{client.name}.pdf",
              type: "application/pdf")
  end
end
```