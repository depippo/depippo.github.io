---
layout: post
title:  "Enhancing a Rails App with Javascript"
date:   2017-02-19 23:49:54 +0000
---


A couple months ago, I built a Rails app for viewing and submitting vegan recipes. It was a perfectly nice little app, built in fairly standard CRUD fashion, but because it was made entirely with Ruby/HTML, it lacked dynamic features. That is to say, every time a link was clicked or a form submitted, a page refresh was necessary. I recently revisited this app with the intention of making it more dynamic, by adding a layer of Javascript functionality. It was an interesting challenge to return to my old code -- code I had originally written without the expectation that I would be adding significant new features in the future. When coding a particular resource's show view, for example, I wasn't anticipating how I would later render this same information via a JSON API.

In the original version of my app, I had an index page of ingredients. Clicking any one of the ingredients would take the user to that particular ingredient's show page, where one could then see all of that ingredient's nutrients (a standard has_many relationship, in other words). The first new feature I set out to add via Javascript was to append those nutrients directly to the ingredients index upon request. The goal was to have a "Show Nutrients" button under each ingredient which, when clicked, would display the corresponding nutrients right there on the page without a redirect.
Figuring that JSON would be the way to go for this, I first ensured that active_model_serializers was in my Gemfile and installed, and then ran 'rails g serializer ingredient' in the console. This gave me a basic 'ingredient_serializer' file like so:

```
class IngredientSerializer < ActiveModel::Serializer
  attributes :id
end
```

Which I then fleshed out slightly to make sure the has_many relationship I needed would be serialized when I made my JSON call:

```
class IngredientSerializer < ActiveModel::Serializer
  attributes :id, :name
  has_many :nutrients
end
```

The html.erb file wouldn't need much modification; essentially just a button to which I could attach the Javascript event.

In my original app I had:

```
  <ul class="list_recipes">
  <% @ingredients.each do |ingredient| %>
      <li class= "list_ingredients"> <%= link_to ingredient.name, ingredient_path(ingredient) %> </li>
  <% end %>
  </ul>
```

Which then became:

```
  <ul class="list_recipes">
  <% @ingredients.each do |ingredient| %>
      <li class= "list_ingredients"> <%= link_to ingredient.name, ingredient_path(ingredient) %>
			<button class="js-shownutrients" data-id="<%= ingredient.id %>">Show Nutrients</button>
			</li>
  <% end %>
  </ul>
```

With this in place, it was time to write some Javascript. I first added a basic attachListeners function to be called when the page loaded:

```
$(document).on('turbolinks:load', function() {
  attachListeners();
});
```

And within that function, I added a click event. When clicked, this stores the ingredient id in a variable, and then calls the showNutrients function, passing in both the event and id variable as arguments:

```
function attachListeners() {
  $(".js-shownutrients").one("click", function(event) {
    var id = $(this).data("id");
    showNutrients(event, id);
  })
}
```

This showNutrients function is where the core functionality is written.

```
function showNutrients(event, id) {
  var list = event.target.parentNode;
  var button = event.target;
  $.get("/ingredients/" + id + ".json").success(function(json) {
    if (json.nutrients.length > 0) {
      $(button).hide();
      json.nutrients.forEach(function(nutrient){
        $(list).append('<li class="json_ingredients">' + nutrient.name + '</li>');
    })}
  });
}
```


To take it line by line, I first set a couple of variables. Since event.target is the button within each ingredient li,  the event.target.parentNode would be the li element of each ingredient. Storing this in the list variable would make it easy to reference later on. I also store the button in a variable so I can easily call the hide function on it. Then I use JQuery to get 'ingredients/:id.json', with the id variable corresponding to the ingredient.id of the button that was clicked. Thanks to the serializer I set up earlier, I know that 'ingredients/:id.json' will return something that looks like this:

```
{
  id: 12,
  name: "almond milk",
  nutrients: [
    {
      id: 10,
      name: "calcium"
    },
    {
      id: 6,
      name: "vitamin E"
    }
  ]
}
```

So when this call is successfully made, json.nutrients will give me the array of nutrients. I first check to see that the given ingredient does indeed have at least 1 nutrient. If so, I hide the "Show Nutrients" button, then loop through all the nutrients in the array and append them to the page, making use of the list variable set earlier so that the new li's are inserted within the their parent ingredient li element. And that's it! The nutrients are now appending to the page without a page refresh.

![jquery walkthrough](https://i.makeagif.com/media/2-19-2017/UdMsrg.gif)

Without going through every line of Javascript I added to the app, I'll just quickly walk through one other new functionality I added. On the recipe show page, I included a basic Rails form for adding user comments:

```
<h5>Submit a comment:</h5>
<%= form_for [@recipe, @comment] do |f|%>
  <p>
    <%= f.label :name %>:
    <%= f.text_field :name %>
    
    <br>
    <%= f.label :content %>:
    <%= f.text_area :content,  cols: "40", rows: "5" %>
  </p>
  <%= f.submit %>
<% end %>
```

In order to get this running without a page refresh, I'd make use of AJAX. In my .js file, I wrote a postComment() function:

```
function postComment(){
  $("form#new_comment").on("submit", function(e){
    e.preventDefault();
    var $form = $(this);
    var action = $form.attr("action")
    var params = $form.serialize()

    $.ajax({
      url: action,
      data: params,
      dataType: "json",
      method: "POST"
    })
    .success(function(json){
      e.preventDefault();
      var comment = new Comment(json);
      var commentLi = comment.renderLi();
      $("#comment_name").val('')
      $("#comment_content").val('')
    })
  })
}
```

Instead of listening for a click event, this function fires when the "new_comment" form is submitted. It makes an AJAX post request, and then uses the JSON response to instantiate a new Comment object. It then calls the renderLi() function on the object, before finally clearing the form fields. This ony works because I've set up an object prototype for Comment, as well as the renderLi() function:

```
function Comment(attributes){
  this.name = attributes.name;
  this.content = attributes.content;
  this.id = attributes.id;
}

Comment.prototype.renderLi = function(){
  var full_comment = "<p class='comment'>" + this.name + " says:<br>" + this.content + "</br> </p>";
  $(".comments_container").append(full_comment)
}
```

Now that I am creating comment objects, I can use custom protype functions to manipulate them. In this case, the function just does a little formatting, adds some tags, and appends the comment to the page. With that, my comments are successfully posting with a refresh:

![jquery walkthrough](https://i.makeagif.com/media/2-19-2017/kkKrbT.gif)

Thanks for reading! The full code for this project is on Github here: https://github.com/depippo/vegan_recipes.


