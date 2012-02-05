---
layout: post
title: "Using javascript: how to do it, live to tell the story and actually enjoy it"
date: 2012-02-04 22:44
comments: true
categories: [javascript, Plone]
---

It is no secret that the relationship between the Plone community (and I imagine the same stands for a large part of the rest of the python world) and javascript is bumpy to say the least. Javascript is considered inferior, ugly, dirty, a scripting language that is an unfortunate necessity. Typically this happens due to these reasons:

* Javascript has been so misused that the web is ridden with bad examples and anti-patterns. Most of us (and I will not exclude myself, at least in the past) have copy-pasted bad examples without ever questioning them or putting the extra effort to understand and find better ways. The good thing is that this is now changing, frameworks exist and they are well documented, and there is now a plethora of good developers who actually are proud and actively advertise being javascript developers.
* It is actually *hard* to see the nice parts of javascript. This is especially true if you don't even try and it is also true that calling something a "self-invoking anonymous function" does not help. But hey, they do exist. Closures are nice. Functions and prototypes are nice. The functional parts of javascript are nice. And a lot of the frameworks that are around nowadays have patterns that manage to be both simple and powerful.
* Purism. The eternal battle of the server vs. client, how curly brackets are so unecessary and how in order to adhere to accessibility everything should work primarily *without* javascript. Oh well, I think I will pass on this one.

In the last few months I have been working on a primarily javascript app. It's a non-Plone related project which I promise to expand upon in later posts but the lessons learned and experience gained apply to javascript written for Plone as well:

1. Let's understand and explore what is there already. I will give an example by means of asyncronous functions, jQuery Deferreds and Promises. Here is the use case: Imagine you want to write a function that executes something asyncronously (fetch from a DB or whatever) and needs to return the result. This is a pretty common pattern in js client code. I will emulate the async nature of the function by using `setTimeout`. Is this what you would do?



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

Now, since jQuery 1.5 jQuery is using the notion of deferreds and promises. Compare the above to the following:

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

Take some time to figure it out and think for a start if you find it cleaner. It is isn't it? First we create a `$.Deferred`. At the end we return the deferred's `promise`. Inside the async part we *resolve* the deffered, which in turn triggers the promise's `done`. Apart from being neater there are a number of additional advantages. To name a couple that a find interesting, a promises's `done` will trigger even after the deferred has been resolved. That is,
if later you did

``` javascript
promise.done(do_something_else);
```

this would trigger immediately since the promise has been fulfilled already. Additionally, you can easily chain deferred and promises, or say if you wanted something to happen after a few promises have been resolved you could do 

``` javascript
$.when(promise1, promise2).then(...);
```

Of course deferreds and promises support handling failures, provide a way to inspect the state of a promise and a few other goodies. Read more about them [here](http://api.jquery.com/category/deferred-object/) and [here](http://api.jquery.com/promise/). You can already start using them on all jQuery objects/functions and also be aware that the success/failure callbacks will be eventually obsolete.

2. Let's stop writing spaghetti code taken from random examples. It does not help us and others understand what we do. It is not maintainable. It is error-prone. My own [jarn.xmpp.core](http://github.com/ggozad/jarn.xmpp.core) although better than average is a good example of what to avoid. The UI and logic *are* (not loosely enough) coupled together. While the use of namespaces gives an impression of structure the different modules are not decoupled and they really require the total to be present to function. When I get the time (or clients) to rewrite it I will.

For this to happen it helps a lot if you have frameworks. The ones I chose, but this depends a lot on what one values, are Underscore and Backbone. I find them both indispensable tools in my daily routine.

* [Underscore](http://documentcloud.github.com/underscore/) gives you all these utilities that you have come to depend upon. Map, reduce, filter, and the likes are the core of the framework. Then there are some additional things I like, especially the function binding and the minimal and powerful template engine.

* [Backbone](http://documentcloud.github.com/backbone/) is my choice for structuring my code and doing MVC. Its primarily use is to support you writing models, collections and views for these. Additionally it gives you a super extensible way with sane defaults to push/pull data to the server that does not get in your way.

Others might choose different tools like [Ember.js](http://emberjs.com/). I like Underscore and Backbone because they are small, they are readable (check out the annotated source, when was the last time you could go through a framework and actually *understand* what it is doing on first read?) but mostly and this is super important THEY DON'T GET IN YOUR WAY. When you want to do things differently they allow you without fighting to do so. Coming from Plone, I can't feel but relieved.

3. Testing. We are pretty good at that in Plone. There is an enormous suite of tests to cover the framework and all respectable add-on products are on it too. But ONLY on the server side right? Then, we sort of act surprised when it fails ;)

I looked at QUnit, JUnit and selenium (in all its variants and collection of python drivers). I found them difficult and disapointing. Then I came across Jasmine and felt in love. Again, simplicity, readability, speed at its best. Powerful mocking. No requirements. I don't need to say more, just grab it and use it.

