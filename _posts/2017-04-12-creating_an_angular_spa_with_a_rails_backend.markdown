---
layout: post
title:  "Creating an Angular SPA with a Rails Backend"
date:   2017-04-11 21:54:21 -0400
---


I recently wrote a Single Page App (SPA) called Vegan Food Finder, which features an Angular frontend and a Rails backend. It was a somewhat lengthy and involved process, but for the purposes of this blog entry, I'd like to focus specifically on how I got Angular communicating with my Rails backend.

I should first mention that I came across a really terrific blog post by Learn.co graduate Jesse Novotny, in which he outlines his process for setting up Devise for use with Rails and Angular: https://www.sitepoint.com/setting-up-an-angular-spa-on-rails-with-devise-and-bootstrap/. I found this very helpful, and I followed a similar pattern to establish the User functionality for my app.
 
### Setting up the Rails Backend

Before digging in to the Angular side too much, I set up a standard Rails app file structure, including models and associations. Although my final app included more models with more attributes, let's just look at Users, Collections, Places, and PlaceCollections (the join model) in a simplified form to get the basic idea.

```
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
  has_one :collection
end
```

```
class Collection < ApplicationRecord
  belongs_to :user
  has_many :place_collections
  has_many :places, through: :place_collections
end
```

```
class Place < ApplicationRecord
  has_many :place_collections
  has_many :collections, through: :place_collections
end
```

```
class PlaceCollection < ApplicationRecord
  belongs_to :place
  belongs_to :collection
end
```

Knowing that every user would need one collection, I added the following to the User model so that each time a new user was created, an associated collection would be as well:

```
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable
  has_one :collection
  after_create :create_collection

  def create_collection
    current_user = User.last
    collection_name = current_user.username
    self.collection = Collection.create(:name => collection_name)
  end
end
```

This was all well and good, but there was still no way for Angular to work with these Rails models. The next step would be to set up an API for my Rails data. Going into the routes.rb file, I added a few new routes :

```
Rails.application.routes.draw do
  devise_for :users
  scope :api do
    get "/collections(.:format)" => "collections#index"
    get "/collections/:id(.:format)" => "collections#show"
    put "/collections/:id/places(.:format)" => "collections#update"
  end
  root 'application#index'
end
```

Aside from devise and the `root 'application#index'` route for Angular, I set up a couple 'get' routes to expose the JSON data, and a 'put' route which will allow Angular to actually add data to our backend. These are all calling actions in the CollectionsController, so let's take a look at that file.

```
class CollectionsController < ApplicationController

  def index
    render json: Collection.all
  end

  def show
    render json: Collection.find(params[:id])
  end

  def update
    collection = Collection.find(params[:id])
    place = Place.find_or_create_by(place_params)
    place.save
    collection.places << place unless collection.places.include?(place)
    collection.save
    user = collection.user
    render json: user
  end

  
private

  def place_params
    params.require(:place).permit(:name)
  end

end
```

The index and show actions are simple and both look like a typical rails app, except that we are rendering the collection data as JSON. The update action is also pretty basic Rails; it find the correct collection according to the params (the :id in the route), finds or creates the place based on the place_params, and then establishes that the place belongs to the collection.

To get the JSON data into a nice and usable format, I then set up ActiveModel serializers for each of my models.

```
class UserSerializer < ActiveModel::Serializer
  attributes :id, :username
end
```

```
class CollectionSerializer < ActiveModel::Serializer
  attributes :id, :name, :places
  
  def places
    object.places.map do |place|
    PlaceSerializer.new(place)
  end

end
```

```
class PlaceSerializer < ActiveModel::Serializer
  attributes :id, :name
end
```

The Place and User serializers just include their respective model's attributes, while the Collection serializer has a bit more logic, becase it needs to render the `has_many :places` relationship.

### Checking the Routes

Okay! Now we have a series of routes that will either give us our rails data in properly formatted JSON, or allow us to post JSON to our backend! We can even navigate to one of these routes to see what our data looks like. With our rails server running, if we go to localhost:3000/api/collections.json (assuming we have some seed data), we'll see something like the following:

```
[
  {
    id: 1,
    name: "john",
    places: [
      {
        id: 1,
        name: "Vedge"
      },
      {
        id: 2,
        name: "Blackbird Pizzeria"
      }
    ]
  }
  {
    id: 2,
    name: "paul",
    places: [
      {
        id: 3,
        name: "HipCityVeg"
      },
      {
        id: 2,
        name: "Blackbird Pizzeria"
      }
    ]
  }
]
```
And if we navigate to a particular collection, such as localhost:3000/api/collections/1.json, we can get a look at not only how our existing data looks, but also the format we will need to use when we POST data to the same route from Angular:

```
{
  id: 1,
  name: "john",
  places: [
    {
      id: 1,
      name: "Vedge"
    },
    {
      id: 2,
      name: "Blackbird Pizzeria"
    }
  ]
}
```

### Integrating with Angular

With all the backend set up out of the way, we can now move on to accessing these routes from the Angular frontend. Let's take a look at a simplified version of the template for one of our views in Angular. 

```
<div ng-repeat="place in searchResults">
  <h2>{{ place.name }}<h2>
  <button type="button" ng-click="vm.saveToCollection(place, user.id)">Save to collection</button>
</div>
```

In this case, the SearchController contains in its scope an array called searchResults, and each object in this array has a name attribute. We've also got access to the current user.id thanks to Devise.

We've added a button that will call our saveToCollection function, passing in the place object as an argument. Let's look at that saveToCollection function:

```
function SearchController($scope, $http, CollectionService) {

var ctrl = this;

ctrl.saveToCollection = function(place, id){
	CollectionService.saveToCollection(place, id)
		.success(function(response){
			console.log(response)
		});
	}

}

angular
.module('vegfinder')
.controller('SearchController', SearchController);
```

All it's really doing is taking the place object and user.id we passed through, and in turn passing them to a Service function that will do the heavy lifting.

```
function CollectionService($http){

    this.saveToCollection = function(place, id){
      var url = '/api/collections/' + id + '/places.json'
      return $http({
        url: url,
        method: 'PUT',
        data: {
          place: place
        }
      })
    }

  }

angular
  .module('vegfinder')
  .service('CollectionService', CollectionService);
```

This function makes use of that api route we set up earlier, and because the place object we've passed through as an argument is in the correct format, we can use that as the data for our call to the API.

Great! We're now able to create and save new instances of our Rails models entirely through the Angular frontend. Let's say we then have a collection view in Angular, in which we'd like to see a list of the places the current user has saved to their collection.
```
<div ng-init="vm.getPlaces(user.id)">
  <div ng-repeat="place in places">
    <h2>{{ place.name }}</h2>
  </div>
</div>
```

We can call a controller function, getPlaces, passing in our user.id to get the appropriate collection.

```
function CollectionController($scope, $http, CollectionService) {

  var ctrl = this;
  $scope.places = [];

  ctrl.getPlaces = function(id) {
    CollectionService.getCollection(id)
    .success(function(response) {
      response.places.forEach(function(item){
        $scope.places.push(item);
      })
    })
  }

}

angular
  .module('vegfinder')
  .controller('CollectionController', CollectionController);
```

This in turn makes use of our CollectionService once again in order to make the actual request to the API. When it gets a response from the CollectionService, it adds the response.places to our CollectionController places array, giving us access to them in our view. The CollectionService function is simpler this time since we're just making a GET request and don't need to pass in any data:

```
function CollectionService($http){

    this.getCollection = function(id){
      var url = '/api/collections/' + id + '.json'
      return $http({
        url: url,
        method: 'GET',
      })
    }

  }

angular
  .module('vegfinder')
  .service('CollectionService', CollectionService);
```
And since we know that API url is returning nicely formatted JSON, Angular will have no trouble rendering each of these places to our template!

That's the basic idea, although of course there's much more that can be done with these methods. The full repo for this project is available at https://github.com/depippo/vegfinder. Thanks for reading!



