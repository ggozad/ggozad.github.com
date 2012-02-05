---
layout: post
title: "Using Javascript: how to do it, live to tell the story and actually enjoy it"
date: 2012-02-04 22:44
comments: true
categories: [Javascript, Plone]
---

It is no secret that the relationship between the Plone community (and I imagine the same stands for a large part of the rest of the Python world) and Javascript is bumpy to say the least. Javascript is considered inferior, ugly, dirty, a scripting language that is an unfortunate necessity. Typically this happens due to these reasons:

* Javascript has been so misused that the web is ridden with bad examples and anti-patterns. Most of us have copy-pasted bad examples without questioning them or putting the extra effort to understand and find better ways. The good thing is that this is now changing, frameworks exist and they are well documented, and there is now a plethora of good developers who actually are proud and actively advertise being Javascript developers.
* It is actually *hard* to see the nice parts of Javascript. This is especially true if you don't even try and it is also true that calling something a "self-invoking anonymous function" does not help. But hey, they do exist. Closures are nice. Functions and prototypes are nice. The functional parts of Javascript are nice. And a lot of the frameworks that are around nowadays have patterns that manage to be both simple and powerful.
* Purism. The eternal battle of the server vs. client, how curly brackets are so unecessary and how in order to adhere to accessibility everything should work primarily *without* Javascript. Oh well, I think I will pass on this one.

In the last few months I have been working on a primarily Javascript app. It's a non-Plone related project which I promise to expand upon in later posts but the lessons learned and experience gained apply to Javascript written for Plone as well:

1. Let's understand and explore what is there already. I will use as an example the most common pattern of calling an asyncronous function with jQuery and doing something on "success". This is such a common operation, but did you know you can improve how you do it by using jQuery's Deferreds and Promises?

To the point: Imagine you want to write a function that executes something asyncronously (fetch from a DB or whatever) and needs to return the result. I will emulate the asyncronous nature of the function by using `setTimeout`. Is this what you would do?

``` javascript
var myFunc = function (value, options) {
    options = options || {};
    var success = options.success;

    setTimeout(function () {
        if (success) {
            success(42 * value);
        }
    }, 1000);

};

myFunc(100, {success: function (res) { console.log(res); }});
```

In version 1.5 jQuery introduced the notion of *deferred* and *promise*. Compare the above to the following code using those:

``` javascript
var myFunc = function (value) {
    var d = $.Deferred();

    setTimeout(function () {
        d.resolve(42 * value);
    }, 1000);
    return d.promise();
};

var promise = myFunc(100);
promise.done(function (res) { console.log(res); });
```

Take some time to figure it out and think for a start if you find it cleaner.... It is, isn't it? First we create a `$.Deferred`. At the end we return the deferred's `promise`. Inside the asyncronous part we *resolve* the deferred, which in turn triggers the promise's `done`. Apart from being neater there are a number of additional advantages. To name a couple that I find interesting, a promises's `done` will trigger even after the deferred has been resolved. That is, if later you did

``` Javascript
promise.done(do_something_else);
```

this would trigger immediately since the promise has been fulfilled already. Additionally, you can easily chain deferreds and promises, or say if you wanted something to happen after a few promises have been fulfilled you could do 

``` javascript
$.when(promise1, promise2).then(...);
```

Of course deferreds and promises support handling failures, provide a way to inspect the state of a promise and a few other goodies. Read more about them [here](http://api.jquery.com/category/deferred-object/) and [here](http://api.jquery.com/promise/). You can already start using them on all jQuery objects/functions and also be aware that the success/failure callbacks will eventually be obsolete. The point of this exercise was to show you that there is a lot you probably don't know about Javascript. It's there for you to explore.

2. Let's stop writing spaghetti code taken from random examples. It does not help us and others understand what we do. It is not maintainable. It is error-prone. My own stuff [jarn.xmpp.core](http://github.com/ggozad/jarn.xmpp.core) although better than average is a good example of what to avoid. The UI and logic *are* (not loosely enough) coupled together. While the use of namespaces gives an impression of structure, the different modules are not decoupled and they really require the total to be present to function. When I get the time (or clients) to rewrite it I will.

For this to happen it helps a lot if you have frameworks. The ones I chose, but this depends a lot on what one values, are Underscore and Backbone. I find them both indispensable tools in my daily routine.

* [Underscore](http://documentcloud.github.com/underscore/) gives you all these utilities that you have come to depend upon. Map, reduce, filter, and the likes are the core of the framework. Then there are some additional things I like, especially the function binding and the minimal and powerful template engine.

* [Backbone](http://documentcloud.github.com/backbone/) is my choice for structuring my code and doing MVC. Its primary use is to support you writing models, collections and views for these. Additionally it gives you a super extensible way with sane defaults to push/pull data to the server that does not get in your way.

Others might choose different tools like [Ember.js](http://emberjs.com/). I like Underscore and Backbone because they are small, they are readable (check out the annotated source and try to remember when was the last time you read through a framework and actually *understood* what it was doing on your first read?) but mostly and this is super important they don't get in your way. When you want to do things differently they allow you to do so without a fight. Coming from Plone, I can't feel but relieved.

3. Testing. Plone has been pretty good at that since the days of 2.5. There is an enormous suite of tests to cover the framework and all respectable add-on products are accompanied by a test suite. Only problem is, these tests rarely cover the client side of things. Javascript becoming an essential part of products and frameworks alike, this should change.

I looked at QUnit, JUnit and selenium (in all its variants and collection of Python drivers). I found them cumbersome and restrictive. Then I came across [Jasmine](http://pivotal.github.com/jasmine/) and fell in love. Jasmine is a BBD 
(behavior-driven development) framework with these advantages:

* Readable, simple and fast.
* Powerful mocking and spies built-in (took me 15 lines to create a mock for an XMPP connection;))
* No requirements on other frameworks, just works with whatever you use.
* It can run either on the browser or headless and there is abundance of drivers for it.

I find that the combination of the above is hard to find together. Use it, it's good!
