---
layout: post
title: "The benefits of addressability in REST web service"
date: 2011-10-21 11:33
comments: true
categories: [rest, web-service]
---
Here are some of my experience after working with `REST` web services for a few years. You may find it useful.

- `The Web` Some people, like me previously, come from relational database won’t understand the benefit of the resource uri when talking about REST web service. They tend to realize more on the primary key of a database record rather than the resource uri. In the json response, the uri usually not being sent out to the client.

{% codeblock lang:javascript %}
{ "id": 1, "name": "Chamnap", "job": "developer" }
{% endcodeblock %}

Most of the cases, the above response should work without any problems. It just arrives when the client API needs to send requests again to that API (maybe it needs more data). Which resource uri? No one remembers it. They need to reconstruct the uri by itself which is a bad thing.

- `Payload` The other thing arises on the connection between resources. Look at request payload:

{% codeblock lang:javascript %}
{ "name": "Chamnap", "job": "developer", "location_id": 1 }
{% endcodeblock %}

Is it a relational database? You would feel with the sense of foreign keylocation_id. It would be better if:

{% codeblock lang:javascript %}
{ "name": "Chamnap", "job": "developer", "location_uri": "http://api.example.com/locations/phnom-penh" }
{% endcodeblock %}

Here, the server should be able to parse the uri, location the correct resource, and save it. The same thing should happen on the GET request of the resource as well.

- The `Location Header` in the response from server. In the case of 201 and 202 status code, this would benefit the client api, since they don’t have to construct the uri to request to see if that resource successfully processed.

{% codeblock lang:javascript %}
Location: http://api.example.com/users/chamnap
{% endcodeblock %}

- `Url Navigation` One good example is pagination link on the resource. Again, the api client doesn’t have to build the uri in order to go next page. The main benefit of not building the uri on the client is that the server api is freely to change it without breaking the existing clients.

{% codeblock lang:javascript %}
{ "id": 1, "from": "http://api.example.com/users/chamnap", "message": "I like it", "next_uri": "http://api.example.com/post/1/comments?page=2" }
{% endcodeblock %}

- `HATEOAS` in simple terms, means response from the server is dynamically bound to the context of the resource. For example, you just send a POST request to make an order. The response from should simple contain the current resource plus the uri to make payment or cancel this order. Doing this way, the client simply go through the uri provided by the server API.

{% codeblock lang:javascript %}
{ "id": 1, "order_date": "2011-11-11", "payment_uri": "http://api.example.com/orders/1/payment", "cancellation_uri": "http://api.example.com/orders/1/cancellation" }
{% endcodeblock %}
