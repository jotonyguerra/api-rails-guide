### Introduction

API's are a common feature of today's web applications. [Google](https://developers.google.com/api-client-library/), [Reddit](https://www.reddit.com/dev/api), [Twitter](https://dev.twitter.com/overview/documentation), and [Github](https://developer.github.com/v3/) all have API's within their software that can be accessed to transfer information to and from external clients. Rails gives you the necessary classes to create your own API. This lesson will go through the basic structure of a client-side API that delivers a universal response known as JSON.

### JavaScript Object Notation

> JSON (JavaScript Object Notation) is a lightweight data-interchange format. It is easy for humans to read and write. It is easy for machines to parse and generate.

[Source](http://www.json.org/)

Since most every programming language has a means of representing name/value pairs and ordered collections, JSON is an optimal format for exchanging information between clients and servers, and between servers. This can be especially useful when interacting between applications written in different languages (say Ruby and JavaScript!). You'll remember you worked with JSON a little during React week and we will be building on that experience. Pay attention to what you want your data structure to look like and how to interact with it to make creating API's as pain free as possible.

### Creating a JSON Endpoint with Rails

To demonstrate how to create a Rails API to send JSON responses, we will modify the code from the lesson [Nested-Routes-Going-Camping][Nested-Routes-Going-Camping]. If you `et get` this lesson, you will see starter code from this assignment that you can walk through with.

The first step is to create a new route for our API endpoint. This will allow us to have both a controller that will send back regular HTML (as we created earlier in the week) and controllers that send back JSON in response to API calls. For demonstration, we will use the Campers model. We will modify our routes.rb file by adding the following code:

```ruby
# config/routes.rb

Rails.application.routes.draw do
...
  namespace :api do
    namespace :v1 do
      resources :campers, only: [:index]
    end
  end
...
end
```

This will create an API path to access our campers information in the JSON format. We add the `api` namespace to let people know that this controller will be returning JSON. We also add the `v1` namespace in order to create a form of version control for the API's we create. As we iterate over versions of our API, this allows us to backwards support our old versions while building out new versions. In the routes, these namespaces simply add the additional slashes to the URL that you will see if you run rake routes. Namespaces also make it possible for us to separate out our regular controllers from our API controllers, since otherwise we would have two controllers and models with the exact same `CampersController` name. When we run rake routes, you will see the new route appear:

```bash
$ rake routes
api_v1_campers  GET /api/v1/campers(.:format)  api/v1/campers#index
```

From here, we need to set up our API controller. In our `app/controllers` folder, we need to create an `api` folder, and then within this folder create a `v1` folder. This is so that Rails will know where to find our namespaced controller based on its conventions. Here, we will create another `CampersController`, this time under the API namespace, that will be responsible for returning the JSON representation of `Campers`, instead of the HTML representation of them. Namespacing things under the API::V1 namespace in our controller class definition allows us to have these multiple `CampersController`s that return different things. It should be noted that adding this API controller will not interfere with the functionality of your current controller because of this namespacing.

Within this controller, we can then add an index method that will return all of our campers back in a JSON format. As you can see, we get all of our campers from the database using a simple `Camper.all` call. Once we have these Camper objects, we can then render them all back as JSON in the response. `render json:` is the Rails way of saying you want to return a collection of data back in the response as a JSON (you may remember seeing a similar thing in our Sinatra API's).

```ruby
# app/controllers/api/v1/campers_controller.rb
class Api::V1::CampersController < ApplicationController
  def index
    render json: Camper.all
  end
end
```

Now, if we visit start our server and visit `localhost:3000/api/v1/campers` we will get the following JSON output (with the exception of the created_at and updated_at which will be specific to when you ran `rake db:seed`:

```no-highlight
[
  {"id":1,"name":"Rovaira","campsite_id":1,"created_at":"2015-12-15T21:22:55.617Z","updated_at":"2015-12-15T21:22:55.617Z"},
  {"id":2,"name":"Jorge","campsite_id":1,"created_at":"2015-12-15T21:22:55.620Z","updated_at":"2015-12-15T21:22:55.620Z"},
  {"id":3,"name":"Brian","campsite_id":1,"created_at":"2015-12-15T21:22:55.623Z","updated_at":"2015-12-15T21:22:55.623Z"},
  {"id":4,"name":"Jesse","campsite_id":2,"created_at":"2015-12-15T21:22:55.625Z","updated_at":"2015-12-15T21:22:55.625Z"},
  {"id":5,"name":"Maribeth","campsite_id":2,"created_at":"2015-12-15T21:22:55.627Z","updated_at":"2015-12-15T21:22:55.627Z"},
  {"id":6,"name":"Kelly","campsite_id":2,"created_at":"2015-12-15T21:22:55.628Z","updated_at":"2015-12-15T21:22:55.628Z"},
  {"id":7,"name":"David","campsite_id":3,"created_at":"2015-12-15T21:22:55.630Z","updated_at":"2015-12-15T21:22:55.630Z"},
  {"id":8,"name":"Phillip","campsite_id":3,"created_at":"2015-12-15T21:22:55.632Z","updated_at":"2015-12-15T21:22:55.632Z"},
  {"id":9,"name":"Kevin","campsite_id":3,"created_at":"2015-12-15T21:22:55.633Z","updated_at":"2015-12-15T21:22:55.633Z"}
]
```

We've successfully created our API endpoint. From here, we can use fetch to seamlessly provide data to our React front end or other developers who might be looking to use our information! Creating this endpoint is key, since it is the source of our client-side API.

### Serialization with JSON and ActiveModel Serializer

When we look at the JSON output above, we can see two potential area for improvement:
- In a larger database, the JSON output will not make sense when taken out of context since there is no reference in the JSON that it is a list of Campers. We should have a top level key that looks something like `campers` to address this.
- We have a lot of information in the JSON response that may not be relevant when working with our application, such as the created_at and updated_at timestamps. It would be great to only return what we need.

This is where Serialization comes into play. The first step is to add the active_model_serializers gem to the Gemfile:

```ruby
# Gemfile
gem "active_model_serializers"
```

ActiveModel::Serializer will also need to be configured with the correct adapter to return your json correctly. In `config/initializer/wrap_parameters.rb`. Add this line to the top of your file.

```ruby
ActiveModelSerializers.config.adapter = :json
```

We will also want to configure your serializer and set the *root* to your JSON file. ActiveModel Serializer can do this for you automatically. It provides lots of nice functionality like this to make creating an API endpoint much simpler and DRYer!

Now, let's clean up our output to display only the information we want, such as the camper id, camper name, and campsite_id to which each camper belongs. Let's first create our `CamperSerializer` model. This will be the serializer for the Camper model. Rails will use its conventions to determine which serializer to use if you do not specify one. Whenever we `render json:` from the camper model Rails will know to use this serializer by default! Pretty cool.

```bash
$ rails g serializer camper
create  app/serializers/camper_serializer.rb
```

Now, we have a serializer in our app folder for our campsite app. The `g serializer` command will create the `serializers` folder for us and it will make a basic template for our serializer, just like you're used to for the `g migration` command. This will give us a starting point for our `CamperSerializer`. The default format looks like this:


```ruby
# app/serializers/camper_serializer.rb
class CamperSerializer < ActiveModel::Serializer
  attributes :id
end
```

It's important to note that the CamperSerializer is inheriting from the class ActiveModel Serializer which is defined in the active_model_serializers gem that we installed. This is what is giving us all of our wonderful serialization behavior. The `attributes` method within this class will determine what attributes are returned as JSON for the model when this serializer is used. You will notice that to start off, this is just the `id`. Visiting `localhost:3000/api/v1/campers` will now result in the following:

```
{"campers":[{"id":1},{"id":2},{"id":3},{"id":4},{"id":5},{"id":6},{"id":7},{"id":8},{"id":9}]}
```

If we want to render back each camper's name and campsite_id in our JSON, we can add these attributes to our camper_serializer.rb file like the following:

```ruby
# app/serializers/camper_serializer.rb
class CamperSerializer < ActiveModel::Serializer
  attributes :id, :name, :campsite_id
end
```

New Output:
```
{
  "campers": [
    {"id":1,"name":"Rovaira","campsite_id":1},
    {"id":2,"name":"Jorge","campsite_id":1},
    {"id":3,"name":"Brian","campsite_id":1},
    {"id":4,"name":"Jesse","campsite_id":2},
    {"id":5,"name":"Maribeth","campsite_id":2},
    {"id":6,"name":"Kelly","campsite_id":2},
    {"id":7,"name":"David","campsite_id":3},
    {"id":8,"name":"Phillip","campsite_id":3},
    {"id":9,"name":"Kevin","campsite_id":3}
  ]
}
```

Now when we visit `localhost:3000/api/v1/campers`, the output will include each camper's name and campsite_id! It will also not include the attributes that we did not specify in the `CamperSerializer` such as `created_at` and `updated_at`. For more ways to modify your serializers and more advanced features, look at the Github Repo for the active_model_serializers [here](https://github.com/rails-api/active_model_serializers).

### Conclusion

Now that our application has a way of sending JSON responses by using an API, we can now use these JavaScript objects in our application, especially in React with fetch. This will allow us to have robust databases and models handled on the Rails side and a React front end that displays all of that wonderful data.

[Nested-Routes-Going-Camping]: /lessons/nested-routes-going-camping
