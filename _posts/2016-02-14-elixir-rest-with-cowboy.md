---
layout: post
title:  "Elixir - REST with Cowboy"
tags:
  - Cowboy
  - Elixir
  - REST
aside: true
---
To be honest, in the last couple of months when I implemented REST interfaces in cowboy, I always implemented the happy path, and I followed the okay-we-will-see principle in case of corner cases. That didn't sound good enough for me so I decided to dig deeper into how to properly implement a REST interface in cowboy. The first thing we need to take care is how many handler module we will need? The fewer is the better of course. We don't want to maintain a bunch of module for a bunch of different HTTP methods. It sounded logical for me that one module for one REST target (product, shop, user, etc).

The next question is how to handle different outcomes, like not found, not authorized, malformed request, etc. So I checked cowboy documentation and I found that in rest modules we need to implement an initialization and handlers for different content types. And for a while I lived with them happily... but they are not enough. There are other functions, callback functions which are good for something.

## Cracking the code

{% comment %}
<div class="separator" style="clear: both; text-align: center;"><a href="https://2.bp.blogspot.com/-Yt0pZUzGvGg/VsBdr_DQIJI/AAAAAAAACoo/4JRd5JDrBHI/s1600/cowboy-rest.jpg" imageanchor="1" style="clear: right; float: right; margin-bottom: 1em; margin-left: 1em;"><img border="0" src="https://2.bp.blogspot.com/-Yt0pZUzGvGg/VsBdr_DQIJI/AAAAAAAACoo/4JRd5JDrBHI/s320/cowboy-rest.jpg" /></a></div>
{% endcomment %}

In order to know how to implement a proper rest interface we need to understand how cowboy implemented rest handlers.
Under the hood cowboy rest handlers are implement via a finite state automaton in the `cowboy_rest` module. Check the [source](https://github.com/ninenines/cowboy/blob/d08c2ab39d38c181abda279d5c2cadfac33a50c1/src/cowboy_rest.erl#L64)! Yeah I know, it is intimidatingly long module. Don't fear it is easy. They are just states of the automata.

As you can see the first interaction with the rest FSM is the invocation of the `rest_init/2`. Here we have opportunity to check the HTTP request and set a default state which will be stored by the underlying rest FSM. Here you can make certain initialisations if you want (get some cookie values, set some of the etc.).

A bit below you can find a good example of handling/calling our callbacks. Check the `known_methods/2` function. It will call our `?MODULE:known_methods/2` function. There is a wrapper function to execute that call, called `call/3`. It gives back `no_call` if there is no callback exported (so default case in a sense), or other values which we are giving back from our callback functions. If you are unsure what a callback function should give back, check the source of the appropriate function which calls our callback. Here you can see that we can say `{:halt, req, state}` in our function, or `{["GET", "POST], req, state}` as a good response. The `next/3` function sets the next state of the rest FSM. Easy, right?

### Flowcharts

Since most of us understand better the graphs, here are the <a href="http://ninenines.eu/docs/en/cowboy/1.0/guide/rest_flowcharts/" target="_blank">REST flowcharts</a> which describe what happens on different return values of the different callbacks.

In the `known_methods` we can check and give back (OPTIONS method for querying them) the possibly methods for a resource. Generally it comes from the rest handler what are implemented what are not, sometimes it depends on the content type of the requests, so we can implement a little logic which generates the possible HTTP methods. In the `allowed_methods` we can restrict the methods for a specific resource (or a specific user given by a cookie). Let us say we have a read-only table in the SQL database, so we don't want to provide POST and PUT methods. In the `is_authorized` we can check permission depending on the current user, if we have such a feature.

We have `content_types_provided` callback for specifying what kind of content type will be generated which of our sub-handler functions. Later you will see how to bind the content type the client wants with the functions we have. In the `content_type_accepted` we can do the same for the incoming data in the request headers.

Let us implement a very simple data access CRUD like service for products. It will have

  * `GET /` for listing all the data in json
  * `GET /:id` for fetching a specific product (results in 200 or 404)
  * `POST /` for creating a <b>new</b> product (results in 201 CREATED)
  * `PUT /:id` for updating a product
  * `DELETE /:id` for deleting a product

You can implement PATCH as a homework. It is a partial update, when the client sends only the changing part of the object.

```elixir
defmodule Store.ProductHandler do
  require Logger

  def init(protocol, _req, _opts) do
    Logger.info("In init/3 #{inspect protocol}")
    {:upgrade, :protocol, :cowboy_rest}
  end

  # init state to be the empty map
  def rest_init(req, _state) do
    {method, req2} = :cowboy_req.method(req)
    {path_info, req3} = :cowboy_req.path_info(req2)
    state = %{method: method, path_info: path_info}
    Logger.info("state = #{inspect state}")
    {:ok, req3, state}
  end

  def content_types_provided(req, state) do
    {[{"application/json", :handle_req}], req, state}
  end

  def content_types_accepted(req, state) do
    {[{\{"application", "json", :"*"}, :handle_in}], req, state}
  end

  def allowed_methods(req, state) do
    {["GET", "PUT", "POST", "PATCH", "DELETE"], req, state}
  end

  # Handling 404 code
  def resource_exists(req, %{:path_info => []} = state) do
    {true, req, state}
  end
  def resource_exists(req, %{:path_info => [id]} = state) do
    Logger.info("Checking if #{id} exists")
    case :ets.lookup(:repo, String.to_integer(id)) do
      [{_id, obj}] ->
        {true, req, Map.put(state, :obj, obj)}
      _ ->
        {false, req, state}
    end
  end

  # Handle DELETE method
  def delete_resource(req, %{:obj => obj} = state) do
    :ets.delete(:repo, obj["id"])
    {true, req, state}
  end

  def handle_req(req, %{:obj => obj} = state) do
    # Handle GET /id
    {Poison.encode!(obj), req, state}
  end
  def handle_req(req, state) do
    # Handle GET /
    response = :ets.tab2list(:repo)
                |> Enum.map(fn({_id, obj}) -> obj end)
                |> Poison.encode!
    {response, req, state}
  end

  # Don't allow post on missing resource -> 404
  def allow_missing_post(req, state) do
    {false, req, state}
  end

  # Handle PUT or POST if resource is not missing
  def handle_in(req, state) do
    {:ok, body, req2} = :cowboy_req.body(req)
    obj = Poison.decode!(body)
    Logger.info("Accepting #{inspect obj}")
    :ets.insert(:repo, [{obj["id"], obj}])
    {true, req2, state}
  end
end
```

(That is what I like in Elixir, I didn't even need to reformat my code after pasting it here ;) ).

You can see, in the `rest_init` I cache the method and the path elements in the state which is a map. In the `handle_req` function we will handle the requests which typically don't have request bodies (GET, HEAD). In the `handle_in` I handle incoming data (POST, PUT).

In the `resouce_exists` I check if a specific record exists in the ETS table. But if we get the index (the list of all objects) we don't have such id and we don't need such a check. If I check the existence of an object, I can cache the object, since it is very probable that I need to fetch that object again. I am caching it in the rest FSM state which will be freed after the rest request is terminated, served.

In the `handle_req` I split the two cases: fetching the index or fetching a specific object. In the `handle_in` we could get here if the request is a POST or PUT. So I get the body of the request, decode the JSON there with `Poision`. It gives back a map, so I can build my tuple which I store in the ETS table. Here we can give back `true` in case of generating a 204 NO CONTENT, or `{true, url}` to generate a 200 OK with body content.

In the `allow_missing_post` by giving back `false` we say that we don't want clients to send POST requests to non-existing resource. So `POST /` is allowed, and a key will be generated, but `POST /99` is not allowed if product with id 99 doesn't exist. In the `delete_resource` we are covering the DELETE method.

From the flowchart you can see how for example a DELETE request can be handled in detail. Like if the object to be deleted, changed since we fetched the object, with the `if-match` header we can check if the request has the same `Etag`, etc. So we have a tons of choices to fully exploit the cowboy rest handlers.

For testing I use the Advanced REST Client Google Chrome extension. I records my http requests, so from the history I can easily replay them.

### Future changes

If you see a very recent version of cowboy, you can see that several thing will be changed. For example `rest_init` and `rest_terminate` are removed, and `init/3` is `init/2` and the function returns are refactored too. Probably we will cover the changes later, but since this is the newer version of cowboy from hex.pm we can safely use this version (1.0.4 at the moment).
