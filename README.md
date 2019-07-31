# Rails URL Helpers

Since Rails by design was meant to be flexible, the result is that there are typically a number of ways to accomplish the same features. Routes are a great example for how this principle operates in a Rails app. In this section we will review how to leverage built-in URL helper methods instead of hard coding route paths into an application (along with why this is a good idea).


## Paths vs Route Helpers

What's a real world difference between using hard coded paths compared with route helper methods? Let's imagine that you have a meeting in NYC, and you want to get from one side of the city to the other. You have a couple different options:

1. Traverse the streets on foot
2. Take a taxi

Walking is like hard coding your route's path. Technically, it can work. However, it's slow, potentially error-prone (one small mistake can lead to the wrong part of town), and, if the meeting location changes, it will require quite a bit of manual work to adjust and walk to the new destination.

Taking a taxi is like using a route helper: you can simply provide the address to the driver and let them navigate the city streets for you. It is faster than walking, and, if the address for the meeting changes while you're en route, it's not as difficult or slow to adjust.

Don't worry if it's still a little fuzzy. Here's an example of what it looks like in code:

* **Hard coded path:** `"/posts/#{@post.id}"`

* **Route helper:** `post_path(@post)`

So why would we want to use route helper methods as opposed to hard coding paths into the application? There are a number of reasons. Below are a few of the key rationales:

* Route helpers are more dynamic since they are methods and not simply strings. This means that if something changes with the route there are many cases where the code itself won't need to be changed at all

* Route helper methods help clean up the view and controller code and assist with readability. *On a side note, you cannot use these helper methods in your model files*

* It's more natural to be able to pass arguments into a method as opposed to using string interpolation. For example, `post_path(post, opt_in: true)` is more readable than `"posts/<%= post.id %>?opt_in=true"`

* Route helpers translate directly into HTML-friendly paths. In other words, if you have any weird characters in your urls, the route helpers will convert them so they can be read properly by browsers. This includes items such as spaces or characters such as `&`, `%`, etc.


## Implementing Route Helpers

To begin, we're going to start with an application that has the MVC set up for `posts`, with `index` and `show` actions currently in place. The route call looks like this:

```ruby
# config/routes.rb
resources :posts, only: [:index, :show]
```

We briefly discussed this `resources` method in the dynamic routing lesson. This will create routing methods for posts that we can utilize in our views and controllers. Running `rake routes` in the terminal will give the following output:

```bash
posts   GET  /posts(.:format)       posts#index
post    GET  /posts/:id(.:format)   posts#show
```

These four columns tell us everything that we'll need in order to use the route helper methods. The breakdown is below:

* **Column 1** - This column gives the prefix for the route helper methods. In the current application, `posts` and `post` are the prefixes for the methods that you can use throughout your applications. The two most popular method types are `_path` and `_url`. So if we want to render a relative link path to our posts' index page, the method would be `posts_path` or `posts_url`. The difference between `_path` and `_url` is that `_path` gives the relative path and `_url` renders the full URL. If you open up the rails console, by running `rails console`, you can test these route helpers out. Run `app.posts_path` and see what the output is. You can also run `app.posts_url` and see how it prints out the full path instead of the relative path. **In general, it's best to use the `_path` version so that nothing breaks if your server domain changes**

* **Column 2** - This is the HTTP verb

* **Column 3** - This column shows what the path for the route will be and what parameters need to be passed to the route. As you may notice, the second row for the show route calls for an ID. When you pass the `resources` method to the `:show` argument, it will automatically create this route and assume that you will need to pass the `id` into the URL string. Whenever you have `id` parameters listed in the path like this, you will need to pass the route helper method an ID, so an example of what our show route code would look like is `post_path(@post)`. Notice how this is different than the `index` route of `posts_path`. Also, you can ignore the `(.:format)` text for now. If you open up the Rails console again, you can call the route helpers. If you have a `Post` with an `id` of `3`, you can run `app.post_path(3)` and see what the resulting output is. Running route helpers in the rails console is a great way of testing out routes to see what their exact output will be

* **Column 4** - This column shows the controller and action with a syntax of `controller#action`

One of the other nice things about utilizing route helper methods is that they create predictable names for the methods. Once you get into day-to-day Rails development, you will only need to run `rake routes` to find custom paths. Let's imagine that you take over a legacy Rails application that was built with traditional routing conventions. If you see CRUD controllers for newsletters, students, sales, offers, and coupons, you don't have to look up the routes to know that you could call the index URLs for each resource below:

* Newsletters - `newsletters_path`

* Students - `students_path`

* Sales - `sales_path`

* Offers - `offers_path`

* Coupons - `coupons_path`


## link_to Method

Our first three tests are currently passing; let's take a look at the lone failure. The failing test ensures that a link from the index page will point to that post's respective show page view template:

```ruby
describe 'index page' do
  it 'links to post page' do
    second_post = Post.create(title: "My Title", description: "My post description")
    visit posts_path
    expect(page).to have_link(second_post.title, href: post_path(second_post))
  end
end
```

This matcher is currently failing since our index page doesn't link to the show page. To fix this, let's update the index page like so:

```erb
<% @posts.each do |post| %>
  <div><a href='<%= "/posts/#{post.id}" %>'><%= post.title %></a></div>
<% end %>
```

Wow, is this 2004? That is some ugly code. Let's use a `link_to` method to clean this up and get rid of multiple `ERB` calls on the same line.

```erb
<% @posts.each do |post| %>
  <div><%= link_to post.title, "/posts/#{post.id}" %></div>
<% end %>
```

This works and gets the tests passing, however it can be refactored. Instead of hardcoding the path and using string interpolation, let's use `post_path` and pass in the post argument.

```erb
<% @posts.each do |post| %>
  <div><%= link_to post.title, post_path(post.id) %></div>
<% end %>
```

This is much better, but to be thorough, let's make one last refactor: Rails is smart enough to know that if you pass in the `post` object as an argument, it will automatically use the ID attribute, so we'll use this implementation code:

```erb
<% @posts.each do |post| %>
  <div><%= link_to post.title, post_path(post) %></div>
<% end %>
```

If you run the tests now, you'll see that they're all still passing.

We're using the `link_to` method to automatically create an HTML `a` tag. If you open the browser and inspect the HTML element of the link, you would see the following:

![Link To](https://s3.amazonaws.com/flatiron-bucket/readme-lessons/link_to.png)

As you can see, even though we never added HTML code for the link –– e.g., `<a href="..."></a>` –– the `link_to` method rendered the correct tag for us.


## Using the :as option

If for any reason you don't like the naming structure for the methods or paths, you can customize them quite easily. A common change is updating the path users go to in order to register for a site. Out of the box, the standard path would be `/users/new`. However, we want something a little more readable, like `/register`.

In order to make this change, let's update the `routes.rb` file, adding the following line:

```ruby
get '/register', to: 'users#new', as: 'register'
```

Now the application lets users navigate to `/register` to sign up, and you, the developer, can utilize your own custom `register_path` route helper throughout the app.


## Summary

Hopefully this lesson shed some light on the beauty of using route helper methods. If you run the tests again after making the above changes, you'll notice something interesting: all of the tests are still passing! If we had hardcoded the URLs in the links in our views, we would have had a major issue: all of our links to the show pages would have broken, along with our Capybara tests. However, by using the built-in helper methods, the links all updated automatically.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/rails-url-helpers-readme'>Rails URL Helpers</a> on Learn.co and start learning to code for free.</p>
