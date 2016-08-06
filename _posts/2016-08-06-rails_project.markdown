---
layout: post
title:  "Rails Project"
date:   2016-08-06 21:12:19 +0000
---

For my Rails Assessment project, I set out to build an app for sharing vegan recipes. (Technically there's nothing stopping you from uploading non-vegan recipes; as of now, there are no validations in place to ensure that all ingredients are vegan. Maybe a futue update!)

Thanks to the handy tools Rails provives for getting a new project started, I was able to get the basic skeleton of the app up and running very quickly. Once my basic models, migrations, controllers, and views were in place (mostly in rudimentary form), it was time to tackle the first real challenge: the form to create new recipes.

The attributes for each recipe evolved a bit as I built the app and played around with it, but I ultimately settled on a recipe form that would allow the user to input :name, :preptime, :servings, :content, and :ingredients. Most of these are very straightforward; :name and :content are just text, and :preptime and :servings are just integers. Each recipe would only need to have one of each of these attributes, so I found that a very standard form_for would suffice for these.

```
<%= form_for @recipe do |f|%>
    <%= f.label :name %>:
    <%= f.text_field :name %>
		
	etc.
```

The trickiest attribute to figure out would be :ingredients, for a couple reasons. Each recipe would need to have several ingredients, and I really wanted to give the user the option of either selecting existing ingredients, or adding new ingredients to the database at the same time they created the recipe. This meant the form would need a checkbox for the ingredients already in the database in conjuction with a field to input new ones, and the create recipe function would need to be able to accept both of these simultaneously. I ended up with a form like the following, with the first line accounting for the existinging ingredients, and then 5 optional fields for adding new ingredients and associating them with the new recipe:

```
    <%= f.collection_check_boxes :ingredient_ids, Ingredient.all.sort_by(&:name), :id, :name %> <br>
    <% 5.times do %>
      <%= f.fields_for :ingredients, @recipe.ingredients.build do |ingredients_fields| %>
        <%= ingredients_fields.label "Add ingredient" %>:
        <%= ingredients_fields.text_field :name %>
      <% end %>
    <% end %>
		
	<%= f.submit %>
	
	
```

For this to work though, I had to write some code in other parts of the app. In the RecipesController, the params would need to be defined to accept nested ingredient attributes:

```
  def recipe_params
    params.require(:recipe).permit(:name, :content, :preptime, :servings,
      :ingredients_attributes => [:name], :ingredient_ids => [])
  end
```

And in the recipe.rb model, I needed to create a custom attributes writer for ingredients. This method first ensures that the create recipe function is not trying to add blank ingredients. Then, it checks whether the ingredient already exists in the database. It adds the existing record to the recipes array of ingredients if it does, or creates a new one if it doesn't:

```
  def ingredients_attributes=(ingredient_attributes)
    ingredient_attributes.values.each do |ingredient_attribute|
      unless ingredient_attribute['name'] == ""
        ingredient = Ingredient.where(name: ingredient_attribute[:name].downcase).first_or_create
        self.ingredients << ingredient
      end
    end
  end
```

And with that, I had a functioning new recipe form, with the option to add ingredients like so:

![](http://i.imgur.com/BadJpVG.png)

With a lot of the heavy lifting out of the way and the core functionality of the app established, I was then free to start adding additional features. The next big challenge was implementing a rating system that would allow users to submit a 1-5 point rating for each recipe. Additionally, I wanted each recipe page to display the average rating it had received, and the ability to view all recipes sorted by their ratings. I created a partial for the ratings form, to be rendered on the recipe show page:

```
<%= form_for [@recipe, @rating] do |f| %>
  <%= fields_for :rating, @recipe.ratings.build do |ratings_fields| %>
    <%= ratings_fields.select :score, (1..5) %>
  <% end %>
  <%= f.submit "Rate this recipe"%>
<% end %>
```

Once I had these ratings, I'd need to do some work with them in my Recipe model to return the information I wanted:

```

  def average_score
    scores = []
    self.ratings.each do |rating|
      unless rating.score.nil?
        scores << rating.score
      end
    end
    scores.inject{ |sum, el| sum + el }.to_f / scores.size
  end

  def self.recipes_with_ratings
    recipes_with_ratings = []
    self.all.each do |recipe|
      if recipe.ratings.count > 0
        recipes_with_ratings << recipe
      end
    end
    recipes_with_ratings
  end

  def self.sort_by_average_score
    recipes = self.recipes_with_ratings
    recipes.sort! { |a, b| b.average_score <=> a.average_score}
  end
```


With these methods in place, my app now had the ability to display average scores for each recipe, and maintain a list of  recipes sorted by those scores:

![](http://i.imgur.com/MbssXm1.png)

The last feature of the app I want to touch on briefly is the photos page. I thought it might be neat if users could upload photos of each recipe as they tried them out. So I added a new model, controller, and views for photos, and updated the schema so that recipes could have_many photos, and photos could belong_to recipes. I also updated the recipe model to accept nested attributes for photos. As for the acutal functionality of uploading photos, I came across a very helpful gem called [carrierwave](https://github.com/carrierwaveuploader/carrierwave), which makes setting this up much easier. With fairly minimal additional code, I soon had a functioning upload form for each recipe's photos page:

![](http://i.imgur.com/bdsDnK3.png)

As with the previous assessment projects on Learn, I had a ton of fun with this one, and feel like I learned a great deal in the process. You can check out the full repository on Github: https://github.com/depippo/vegan_recipes.

