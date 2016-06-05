---
layout: post
title:  "Product Reviews - Sinatra Assessment Project"
date:   2016-06-05 19:33:10 +0000
---

The Sinatra Assessment Project on Learn.co tasks students with creating a CRUD (Create Read Update Delete) app. I wasn't initially sure just what sort of content I wanted my app to manage, so I described the parameters of the project to my wife and asked her what she might use this type of app for. She suggested something to keep track of her collection of beauty products, so that is what I set out to build.

The first thing to do was determine what models my app would need, and then to create the schema. I decided this app would need 3 models: Users, Products, and Reviews. From here, before doing any real coding, I could set up most of the file structure of the app.

```
├── app
│   ├── controllers
│   │   ├── application_controller.rb
│   │   ├── product_controller.rb
│   │   ├── review_controller.rb
│   │   └── user_controller.rb
│   ├── models
│   │   ├── product.rb
│   │   ├── review.rb
│   │   └── user.rb
│   └── views
│       ├── index.eb
│       ├── layout.erb
│       ├── products
│       │   ├── index.erb
│       │   └── show.erb
│       ├── reviews
│           ├── edit.erb
│           ├── index.erb
│           ├── new.erb
│           └── show.erb
│       └── users
│           ├── index.erb
│           ├── login.erb
│           └── signup.erb
```

With this in place, my next step was to determine the associations my models would need, and write them into the respective model.rb files like so:
```
class Review < ActiveRecord::Base
  belongs_to :user
  belongs_to :product
end
```
I was now ready to create migrations for each of my models. After a bit of deliberation, I settled on the attributes each model would need, wrote up the migration for each one, and then ran them with rake. My project now had a db folder and a schema.rb file that looked like this:
```
ActiveRecord::Schema.define(version: 20160603020841) do

  create_table "products", force: :cascade do |t|
    t.string "name"
  end

  create_table "reviews", force: :cascade do |t|
    t.string  "content"
    t.integer "user_id"
    t.integer "product_id"
  end

  create_table "users", force: :cascade do |t|
    t.string "username"
    t.string "password_digest"
  end

end
```
With the db set up, I then started work on the controllers, beginning with application_controller.rb. This would set some general parameters for the project and contain the controller action for the main index page.

```
class ApplicationController < Sinatra::Base

  configure do
    set :public_folder, 'public'
    set :views, 'app/views'
    enable :sessions
    set :session_secret, "Kiwi"
  end

  get '/' do
   erb :'/index'
  end
  
end
```
The other controllers would need to be more complex. I started by quickly building a skeleton of the review controller, including the get/post requests I knew I would need, and establishing connections to my views (even though I hadn't built those just yet).

```
class ReviewController < ApplicationController
 
   get "/reviews" do
     erb :"/reviews/index"
   end
 
   get "/reviews/new" do
     erb :"/reviews/create_review"
   end
 
   post "/reviews" do
     @review = Review.create(content: params[:content])
     redirect to "/reviews"
   end
 
 end 
```
I repeated this process for the Product and User Controllers, and then began to fill in some code for my placeholder view files. I started to make use of shotgun here, to verify that my routes were working correctly. Once they were, I started to build out the core functionality of the app, the ability to create a review. Again, I chose to start by building this in the most rudimentary way I could, just to get it working, before building it out into the more polished version I ultimately wanted. At first, this meant that my create_review.erb view looked something like this:

```
 <form action="/reviews" method="POST">
  
  <h2>Enter the name of the product:</h2>
  <input type="text" name="product_name">
 
  <h2>Enter your review here:</h2>
  <input type="text" name="content">
  
  <input type="submit" value="submit">
 
 </form>
```
 
I continued in this fashion for the rest of my views, updating the corresponding controllers as necessary, with the goal of getting basic functionality established first. As I built these basic connections for the app, and played around with it using shotgun, I tried to think of ways to improve the app. For example, I decided that my create_review form should really give users the option to select a product that had already been reviewed by another user. By adding the following snippet of code, I was able to add a drop-down menu to the form:

```
<h3>Or select an existing product from the menu:</h3>
<select name="Product Name">
  <option selected disabled hidden style='display: none' value=''></option>
  <% @products.each do |product| %>
    <option value="<%= product.name %>"><%=product.name%></option>
  <% end %>
</select>
```
This led me to thinking that, if some products were going to have multiple reviews by different users, it would be fun if the products index view showed how many reviews were associated with each project, and allowed you to click through to see all the those reviews:

```
<h2>Click on a product to see all of its reviews.</h2>
<a href="/reviews"><input type="submit" value="Return to main reviews page"></a>

<% @products.each do |product| %>
  <% count = product.reviews.count %>
  <h3> <a href="/products/<%=product.slug%>"><%=product.name%></a> - <%=product.reviews.count%> <%= count == 1 ? ("review") : ("reviews") %>  </h3>
<%end%>

```
Continuing in this fashion, my app really started to take shape. On top of the basic functionality of being able to create, read, update, and delete content though, the app also needed validations. For one, I needed to make sure that only valid users could create content, and crucially, that users could only update and delete their own reviews. I added a couple basic helper methods to verify the user:

```
class Helpers

  def self.current_user(session)
    @current_user ||= User.find_by(id: session[:user_id]) if session[:user_id]
  end

  def self.is_logged_in?(session)
    !!session[:user_id]
  end

end
```
I then employed these throughout the app to ensure that no one would be able to alter other users' content. For example, in show_review.erb, I was able to make the edit and delete links viewable only to the user who had created the review.

```
<h2>Review of <%= @review.product.name %> </h2>
<h3>by <%= @review.user.username %> </h3>
<p> <%= @review.content %> </p>

<% if @review.user_id == Helpers.current_user(session).id %>
  <a href="/reviews/<%=@review.id%>/edit"><input type="submit" Value="Edit Review"></a>
  <br></br>
  <form action="/reviews/<%=@review.id%>/delete" method="POST">
    <input type="hidden" id="hidden" name="_method" value="DELETE">
    <input type="submit" Value="Delete Review">
  </form>
  <%end%>
```
And for good measure, just in case someone naughtily tried to navigate directly to /reviews/:id/edit, I added a validation in the get request as well.

```
  get '/reviews/:id/edit' do
    if Helpers.is_logged_in?(session)
      @review = Review.find(params[:id])
      if @review.user_id == Helpers.current_user(session).id
        erb :'/reviews/edit_review'
      else
        redirect to "/reviews"
      end
    else
      redirect to "/login"
    end
  end
```
I also needed to make sure that users couldn't create empty reviews, or reviews of nameless products, and I wanted to alert them to their error if they attempted to do such a thing. Adding a condition to my post request, `if params[:content] != "" && params["Product Name"] !=""` let me make sure empty data wasn't being created, and adding session messages would allow me to notify the user.

```
  post "/reviews" do
    if params[:content] != "" && params["Product Name"] !=""
      @review = Review.create(content: params[:content])
      @review.user = Helpers.current_user(session)
      @review.product = Product.find_or_create_by(name: params["Product Name"])
      @review.save
      session[:success_message] = "Successfully created review."
      redirect to "/reviews/#{@review.id}"
    else
      session[:failure_message] = "Please enter the name of product you're reviewing, and type a review in the text box."
      redirect to "/reviews/new"
    end
  end
```
The session message would then need to be added to the corresponding get request:

```
  get "/reviews/new" do
    if Helpers.is_logged_in?(session)
      @products = Product.all
      @failure_message = session[:failure_message]
      session[:failure_message] = nil
      erb :"/reviews/create_review"
    else
      redirect to "/login"
    end
  end
```
And then displayed on the appropriate view if neccesary:

```
<% if @failure_message %>
  <%= @failure_message %>
<% end %>
```
And with that, I now had a fully functioning CRUD app! As with the previous assessment project, I had a lot of fun making this. I'm finding it extremely rewarding to build something of my own from the ground up. The full repo of this project is available on github [here](https://github.com/depippo/sinatra-product-reviews).

