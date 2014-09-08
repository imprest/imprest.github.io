---
layout: post
title:  "Cowboy hook to support angularjs html5Mode"
date:   2014-09-05 22:33:33
comments: true
tags: erlang cowboy angularjs
---

While using the [$location][$location] service with html5Mode set to true,
on a refresh to say a url like `http://myawesomeapp.com/about`
cowboy would give a 404 since the url or routing was suppose to be handled by 
the front-end framework i.e. angularjs.

The following [cowboy on response hook][cowboy-hooks] resolves the issue for me.
First add the hook when you start cowboy.

{% highlight erlang %}
%% app.erl
cowboy:start_http(my_http_listener, 100,
    [{port, 8080}],
    [
        {env, [{dispatch, Dispatch}]},
        {onresponse, fun ?MODULE:custom_404_hook/4}
    ]
).
{% endhighlight %}

Then in your `custom_404_hook` just return `index.html` so angular takes care
of the routing on the front-end.

{% highlight erlang %}
%% app.erl
custom_404_hook(404, Headers, <<>>, Req) ->
    % Assuming index.html is at priv/app/index.html
    Path = filename:absname("app/index.html", code:priv_dir(app)),
    {ok, Body} = file:read_file(Path),
    Headers2 = lists:keyreplace(
                   <<"content-length">>, 1, Headers,
                   {<<"content-length">>, integer_to_list(byte_size(Body))}),
    Headers3 = lists:keyreplace(
                   <<"content-type">>, 1, Headers2,
                   {<<"content-type", <<"text/html">>}),
    {ok, Req2} = cowboy_req:reply(200, Headers3, Body, Req),
    Req2;
custom_404_hook(_, _, _, Req) ->
    Req.
{% endhighlight %}

Now going to `http://myawesomeapp.com/about` should work :)

[$location]: https://docs.angularjs.org/guide/$location
[cowboy-hooks]: http://ninenines.eu/docs/en/cowboy/1.0/guide/hooks/

