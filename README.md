## Objectives

  1. Understand the value of removing logic from unexpected places
  2. Know the definition of presentation logic

## Software is hard
Making complex web applications isn't easy. It requires you to have knowledge of many different languages and frameworks. As developers, we need to do everything possible to give ourselves a fighting chance.

It's possible for us to create a web application using a single file containing all of our code. As that file gets bigger and bigger, adding new features and debugging problems becomes nearly impossible. What are our options then? This is where Model-View-Controller steps in to save the day.

## MVC to the rescue
Model-View-Controller provides us with a guidebook on how to separate our code. Each part of this pattern handles a specific responsibility for the application. This splitting of responsibility is often referred to as **Separation of Concerns**. Each part of MVC is only concerned with one thing. Let's look at how these concerns are assigned. 

* **Model:** This handles the data for our application. Anything to do with data is placed here: persistence, retrieval and calculations. This is often referred to as Business Logic. In Rails these are the model classes.
* **View:** This is the interface the user sees. The user can interact with the application.  For example, the user can click on a links on a page or fill out forms.  In Rails, the code for our views live in templates, often referred to as ERB files.
* **Controller:** This is the middleman between the model and the view. Its job is to handle/route requests, figure out what data to load and pick the correct view.

## Eating at MVC Diner
Think of your favorite restaurant.  Got it?  Great. Your restaurant is a great example of how the real world breaks up work into specific roles. Let's see how these roles fit into MVC.

### Model
The cook prepares the food for the customers. They don't talk to the customer directly. The cook just makes the food they are told to make and gives it to the waiter. This is the model part of MVC.


### View
The customers are only interested in eating. They place an order with the waiter and eat the food once it's brought to their table. This is the view part of MVC.


### Controller
The waiter/waitress takes the customer's order to the cook and then they pick up the food from the cook once it's ready. This is the controller part of MVC.


## But wait, how does this relate to software?

- Viewing a menu at a restaurant is like sitting down at your computer and visiting up a news website.

- When you decide to order Spaghetti (**yum!**) it's the same as clicking on a title for an article.  You're interacting with the view.

- The waiter, like a controller, routes the requst.

- The cook is then responsible for making the food.  Just as with the models, this is where the magic happens.  Data is retrieved and persisted to the database, just like raw ingredients are turned into your delicious Spaghetti.


- The waiter picks up your Spaghetti and sets it on your table just like the controller displays the selected article to your web browser.


Now imagine if the waiter and the cook were the same person.  Roles and responsibilities would be mixed up.  Chances are that one person will have hard time juggling all their tasks.  Without this separation, not a lot would get done or it will be very slow. Having one person do everything all at once can cause chaos and isn't very efficient. This isn't much different from writing software. If we try to put all of our code in one file, productivity will slow to a crawl.

## Putting it all together
Now we know the basics of how we might organize our code. You've started creating your next big idea and maybe your still not sure what goes where. Let's look at some examples to drive the idea home. In our application, we may want to show two different links based on if a user is signed in or not.

```ruby

<% if user_signed_in? %>
  <a href="profile">Profile</a>
<% else %>
  <a href="sign_in">Sign In</a>
<% end %>

```

This logic clearly handles how the view is presented to the user. We would call this **presentation logic**. This probably makes sense, but where do we draw the line? How do we decide if code belongs in a view?  Take a look at the next example.


```ruby
<% posts = Posts.all %>
<ul>
	<% posts.each |post| %>
		<li><% post.text %></li>
	<% end %>
</ul>

```
In this example we fetch all the posts from the database and created a list to view the text of each post. The problem with this code is we are directly accessing the database in the view.  But wait, isn't that the job for the controller?  Correct!  The controller is resonsible for figuring out what data to load so let's move this code.

```ruby
class PostsController < ApplicationController
	def index
		# Controller determines what data to load
		@posts = Posts.all
	end
end

```

Great, the controller gave the posts to the view.  Now the view can continue displaying the posts without having to access the database.

Let's take a look at one more example. When viewing a post, I'd like to display how many likes a post has received.

```ruby
# Post
def show
	@posts = Post.find(params[:id])
	@like_count = 0
	
	@post.comments.each do |comment|
	    @like_count += 1 if comment.liked?
	end
end

```

This seems to make sense in the controller but this code is actually **Business Logic**. Business logic is the heart of an application.  The controller's job is just to figure out what model to load and what view to show.  The waiter wouldn't tell the cook how to make the Spaghetti. Since this code in the previous example has to do with posts, let's put it in the post model.

```ruby
class Post < ActiveRecord::Base
	def like_count
		like_count = 0
		self.comments.each do |comment|
	    	like_count += 1 if comment.liked?
		end
		like_count
	end
end

```
Not only have we moved the code to a place that makes more sense, we are also able to reuse it in other parts of the application. That's a big win for us.

## Final Thoughts
As we create more and more code, we may find ourselves creating a lot of logic in the our views. This logic might still be Presentation Logic but it can quickly get hard to understand. We will want to move the logic eventually to somewhere else. Rails helpers are a good place to start. Helpers allow use to share this logic accross multiple views.