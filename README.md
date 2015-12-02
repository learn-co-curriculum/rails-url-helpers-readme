# Rails URL Helpers

## Objectives

1. Describe why route helpers are useful (relying on abstractions such as methods as opposed to literring our application with URL data)
2. Use rake routes to find the name of a route
3. Predict route names based on rails routing conventions
4. Use link_to to generate an <a>
5. Use _path and _url helpers to generate URLs
6. Name routes with :as option
7. Generate URLs with route helpers that require variables.


## Notes

Given a posts#index and a posts#show, they should link between each other

we could use `<a href="/posts/#{@post.id}">` but thats very low level.

Let's start by having posts#index link to individual posts show pages.

link_to instead of a href but lets still use interpolation.

Then let's introduce the idea of named routes and instead of hardcoding URLs we'll let our URL Helpers generate them.

what are the _path / _url helpers and when / where in teh stack can you use them (views/controllers yes, models no)

Why? Maintainability, relying on abstractions.

rake routes

how routes get names, the default

works great for posts_path , bad for /posts/:id

introduce the :as option to explictly name a route

common error of trying to generate a route with a variable without providing an argument for that route variabel value.

passing data to dynamic routes from lowest level to highest level

using post_path(:id => @post.id)

using post_path(@post.id)

using post_path(@post)

<a href='https://learn.co/lessons/rails-url-helpers-readme' data-visibility='hidden'>View this lesson on Learn.co</a>
