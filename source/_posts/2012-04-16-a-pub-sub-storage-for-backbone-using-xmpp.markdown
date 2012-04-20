---
layout: post
title: "A Pub-Sub storage for Backbone using XMPP"
date: 2012-04-16 16:54
comments: true
categories:
---

[Backbone.js](http://documentcloud.github.com/backbone/) is a very popular framework for building web applications. Building on top of the excellent [Underscore.js](http://documentcloud.github.com/underscore/), an essential tool for js development, it provides the basic app-building blocks, *Models*, *Collections* and *Views* for them as well as a simple *Router* on top.

What I enjoy the most about Backbone, is its simplicity. It provides me with what I need to be productive, no more no less. Its assumptions are limited to the essential ones leaving me the freedom to be creative and to not have to fight with it when I want to differ in my approach.

In order to connect models and collections with the server-side of things, Backbone's default approach is making RESTful Ajax requests using a CRUD approach. All the operations that read from or persist data to the server are delegated to a *sync()* function that translates *create*, *read*, *update*, *delete* to *PUT*, *GET*, *POST*, *DELETE*. It then goes on to make these calls through jQuery, and the job is done.

The beauty of it, is that one only needs to override `sync` to provide an alternative storage. A popular example is [Backbone.localStorage](https://github.com/jeromegn/Backbone.localStorage) which, well, persists to the browser's *localStorage*.

At Riot AS we have been building a team collaboration web application centered around XMPP. I will post soon in detail about the architecture on which the app is based, but in short we have *completely* replaced Ajax requests in favor of doing all client-server as well as client-client communications through XMPP. As a result, we have been needing a storage adapter to XMPP, and [Backbone.xmpp](https://github.com/ggozad/Backbone.xmpp) was born.

**Backbone.xmpp** is a drop-in replacement for Backbone's RESTful API, allowing models/collections to be persisted on XMPP PubSub nodes. Naturally, *Collections* are mapped to *Nodes*, whereas *Models* to *Items* on these nodes. This is as easy as overriding its `sync()` function and providing an instance of `PubSubNodeStorage` on the `node` attribute of the collection. For example, if we had a collection of type `Library` consisting of models of type `Book` the collection would be
{% codeblock lang:javascript %}
var MyCollection = Backbone.Collection.extend({
    sync: Backbone.xmppSync,
    model: MyModel,
    initialize: function () {
        this.node = new PubSubNodeStorage('mymodels'),
    },
    ...
});
{% endcodeblock %}

