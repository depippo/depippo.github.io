---
layout: post
title:  "Using the Foursquare API with Angular"
date:   2017-04-13 14:21:30 +0000
---


I recently came across an article -- it's a few years old now, but still relevant I think -- about how a very wide swath of the internet relies on data from the Foursqaure API: https://techcrunch.com/2013/03/29/the-internet-needs-foursquare-to-succeed/. In addition to the wealth of data it offers, a big part of what makes the Foursqaure API so appealing to me is its excellent documentation. I spent some time working with the API and integrating it with an Angular app, so I'd like to walk through my process.

The first step is to register at https://developer.foursquare.com, click Create New App, and receive your client_id and client_secrets. From there, Foursquare has a very handy guide under the heading "Endpoints" breaking down all of the URLs you can use to retrieve data from the API.

![Imgur](http://i.imgur.com/BJfUsjp.png)

If you click one of the endpoints, like Venues, for example, you will see further details, such as required and optional parameters, as well as the feature I love the most: the Try It Out button. I still find [Postman](https://www.getpostman.com/) invaluable for exploring APIs, but this is like a lite version of Postman you can play around with right in your browser. You can see exactly what the response will be for any endpoint you might want to hit in your app before writing any code.

![Imgur](http://i.imgur.com/s1Uav49.png)


For my project, starting with a basic Angular file structure, I created a service to make the API call.

```
function SearchService() {

  this.searchVenues = function(city){
    var baseUrl = "https://api.foursquare.com/v2/venues/explore?";
    var client_id = CLIENT_ID;
    var client_secret = CLIENT_SECRET;
    var url = baseUrl + "client_id=" + client_id + "&client_secret=" + client_secret + "&v=20160815&query=vegan&venuePhotos=1&near=" + city;

    return $.getJSON(url, function(){
      console.log("success");
    })
  }

}

angular
  .module('vegfinder')
  .service('SearchService', SearchService);
```

(CLIENT_ID and CLIENT_SECRET should be replaced with the actual values recevied from Foursquare, or references to  variables containing them.) I was able to construct this URL pretty easily by playing around with the Try It Out feature and settling on an endpoint that gave me the results I wanted. I hard-coded the 'vegan' search term into the URL, but left the location as a variable, so whenever this searchVenues function runs, it will retrieve vegan restaurants near whatever location is passed in as an argument. (I ran into some hurdles with [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS) when trying to hit Foursquare endpoints via an $htttp.get request, but I found that using the JQuery method $.getJSON to be an effective workaround.)


I also set up a controller to call the service function and handle the response.

```
function SearchController($scope, SearchService) {

  var ctrl = this;

  ctrl.venueSearch = function(city) {
    SearchService.searchWithin(city)
    .done(function(response) {
      $scope.searchResults = [];
      response.response.groups[0].items.forEach(function(item){
        $scope.searchResults.push(item);
      });
    })
  }

}

angular
  .module('vegfinder')
  .controller('SearchController', SearchController);
```

Again, I was able to determine precisely what I wanted to push into my searchResults array mostly by exploring the Try It Out feature on the Foursquare page. That said, throwing in a console.log(response) after the service function is called and checking it out in the browser developer tools can be very helpful as well.

And then to actually call this function as a user of the app might, I created a simple form in my search.html template. 

```
<form ng-submit="vm.venueSearch(vm.city)" remote="true" id="search-form">
  <input type="text" placeholder="Enter city, state" ng-model="vm.city" />
</form>
```

In my routes.js file, I've defined the SearchController as vm. This form binds the user input to vm.city, and when the form is submitted, passes that as a variable to the venueSearch controller function.

Now to display the search reults in the view:

```
<div ng-repeat="item in searchResults">
  <h4>{{ item.venue.name }}</h4>
  <img ng-src="{{ item.venue.photos.groups[0].items[0].prefix}}128{{item.venue.photos.groups[0].items[0].suffix }}" >
  <h5> {{ item.venue.location.address }} </h5>
  <h5> {{ item.tips[0].text }} </h5>
</div>
```
This is simply iterating over the searchResults arrray set up earlier in the Controller, and displaying select attributes for each item. Each of these objects contains a lot more data than this, so this is just a small sampling as an example. I'll admit that getting the image to display properly threw me for a loop initially, as it is not included in the response as a single string like you may expect. Both a prefix and suffix are returned as separate fields, and they need to be concatanated and given a size (128 in this case).

With that, I had a simple but functional search feature in place:

![Imgur](http://i.imgur.com/QGneDwQ.png)

There's a *ton* more you can with this API, but I found this to be a fun jumping off point, and a solid foundation for more complicated implementations. Thanks for reading!


