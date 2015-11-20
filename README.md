# Rails URL Helpers

Since Rails by design was meant to be flexible, the result is that there are typically a number of ways to accomplish the same features. Routes are a great example for how this principle operates in a Rails app. As you have already seen in our Capybara feature specs, we have the ability to hardcode paths or to use view helper methods. In this section we will extend this knowledge and walk though both options and how they can be used in the implementation code itself.


## Paths vs Route Helpers

So why would we want to use route helper methods as opposed to hard coding paths into the application? There are a number of reasons, below are a few of the key rationales:

* Route helper methods help clean up the view and controller code and assist with readability. On a side note, you cannot use these helper methods in your model files.

* It's more natural to be able to pass arguments into a method as opposed to using string interpolation. For example, using ```post_path(post, opt_in: true)``` is more readable than using ```"posts/<%= post.id %>?opt_in=true"```

* Route helpers are more dynamic since they are methods and not simply strings

* Route helpers translate directly into HTML friendly paths


## Implementing Route Helpers

To begin, we're going to start with an application that has the MVC setup for ```posts```, with ```index``` and ```show``` actions currently in place. The route call looks like this:

```ruby
# config/routes.rb
resources :posts, only: [:index, :show]
```

We briefly discussed this ```resources``` method in the dynamic routing lesson, this will create routing methods for posts that we can utilized in our views and controllers. By running ```rake routes``` in the terminal it will give the following output:

```
posts   GET  /posts(.:format)       posts#index
post    GET  /posts/:id(.:format)   posts#show
```

These four columns tell us everything that we'll need in order to use the route helper methods. The breakdown is below:

* **Column 1** - This column gives the prefix for the route helper methods. In the current application ```posts``` and ```post``` are the prefixes for the methods that you can use throughout your applications. The two most popular method types are ```_path``` and ```_url```. So if we want to render a relative link path to our posts' index page the method would be ```posts_path``` or ```posts_url```. The difference between ```_path``` and ```_url``` is that ```_path``` gives the relative path and ```_url``` renders the full URL.

* **Column 2** - This is the HTTP verb

* **Column 3** - This column shows what the path for the route will be and what parameters need to be passed to the route. As you may notice the second row for the show route calls for an ID. When you pass the ```resources``` method the ```:show``` argument it will automatically create this route and assumes that you will need to pass the id into the URL string. Whenever you have id parameters listed in the path like this you will need to pass the route helper method an id, so our show route would look like this: ```post_path(@post)```. Notice how this is different than the index route of ```posts_path```.

* **Column 4** - This column shows the controller and action with the syntax of ```controller#action```.

One of the other nice things about utilize route helper methods is that they create predictable names for the methods. Once you get into day to day Rails development you will only need to run ```rake routes``` to find custom paths. Let's imagine that you takeover a legacy Rails application that was built with traditional routing conventions. If you see CRUD controllers for: newsletters, students, sales, offers, and coupons; you don't have to lookup the routes to know that you could call the index URLs for each resource below:

* Newsletters - ```newsletters_path```

* Students - ```students_path```

* Sales - ```sales_path```

* Offers - ```offers_path```

* Coupons - ```coupons_path```


## link_to Method

All of our tests are currently passing, let's create a new Capybara spec. The scenario we will be building is to ensure that a link from the index page will point to that post's respective show page view template. The scenario is below:

```ruby
scenario 'index page links to post page' do
  second = FactoryGirl.create(:second_post)
  visit posts_path
  expect(page).to have_link(second.title, href: post_path(second))
end
```

This matcher will fail since our index page doesn't currently link to the show page. To fix this let's update the index page like so:

```ERB
<% @posts.each do |post| %>
  <div><%= link_to post.title, "/posts/#{post.id}" %></div>
<% end %>
```

This works and gets the tests passing, however it can be refactored. Instead of hardcoding the path and using string interpolation, let's using ```post_path``` and pass in the post argument.

```ERB
<% @posts.each do |post| %>
  <div><%= link_to post.title, post_path(post.id) %></div>
<% end %>
```

This is much better, but to be thorough, let's make one last refactor: Rails is smart enough to know that if you pass in the ```post``` object as an argument, it will automatically use the ID attribute, so we'll use this implementation code:

```ERB
<% @posts.each do |post| %>
  <div><%= link_to post.title, post_path(post) %></div>
<% end %>
```

If you run the tests now you'll see that they're all still passing.

We're using the ```link_to``` method to automatically create an HTML ```a``` tag. Now all of the tests are passing. If you open the browser and inspect the HTML element of the link you would see the following:

![Link To](http://reif.io/lib/flatiron/link_to.png)

As you will see, even those we never added HTML code for the link, such as: ```<a href="..."></a>``` the ```link_to``` method rendered the correct tag for us.


## Using the :as option

If for any reason you don't like the naming structure for the methods or paths you can customize them quite easily. Let's say we want to change the show path from:

```
/posts/42
```

to:

```
/post/42
```

In order to do this, let's update the route's file like so:

```ruby
get '/post/:id', to: 'posts#show', as: 'post'
```

Now you will notice the beauty of using route helper methods. If you run our tests you will notice something interesting: all of the tests are still passing! If we had hardcoded the URLs in the links in our views we would have had a major issue: all of our links to the show pages would have broken, along with our Capybara tests. However, by using the built in helper methods the links all updated automatically.