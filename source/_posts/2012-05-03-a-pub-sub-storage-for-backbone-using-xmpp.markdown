---
layout: post
title: "A Pub-Sub storage for Backbone using XMPP"
date: 2012-05-03 16:54
comments: true
categories: [Javascript, XMPP, backbone.js]
---

[Backbone.js](http://documentcloud.github.com/backbone/) is a popular framework for building web applications with javascript. Building on top of the excellent [Underscore.js](http://documentcloud.github.com/underscore), it provides the basic app-building blocks, *Models*, *Collections* and *Views* as well as a simple *Router*.

In order to connect models and collections with the server-side of things, Backbone's default approach is CRUD through RESTful Ajax requests. All the operations that read from or persist data to the server are delegated to a `sync()` function that translates *create*, *read*, *update*, *delete* to *PUT*, *GET*, *POST*, *DELETE* through jQuery.

What I enjoy the most about Backbone is its simplicity, how it gives me what I need to be productive, no more no less. In the case of the storage layer, one only needs to override `sync` in order to use an alternative. A popular example is [Backbone.localStorage](https://github.com/jeromegn/Backbone.localStorage) which, persists to the browser's *localStorage*.

I have been part of building a team collaboration web application centered around XMPP at Riot AS. I will post details about the architecture on which the app is based, but in short we have *completely* replaced Ajax requests in favor of doing all client-server as well as client-client communications through XMPP. To do so, the need arose of a storage adapter to XMPP, and [Backbone.xmpp](http://ggozad.github.com/Backbone.xmpp) was born.

**Backbone.xmpp** ([GitHub](http://ggozad.github.com/Backbone.xmpp))is a drop-in replacement for Backbone's RESTful API, allowing models/collections to be persisted on [XMPP Pub-Sub nodes](http://xmpp.org/extensions/xep-0060.html). Naturally, Collections are mapped to *nodes*, whereas Models to the *items* of these nodes. Additionally, it listens for events on these nodes, receiving and propagating real-time updates of the models/collections from the server. Apart from Backbone, it depends on [Strophe](http://github.com/metajack/strophejs) as well as my [Strophe plugins](http://ggozad.github.com/strophe.plugins) for Pub-Sub.

For example, if you had a collection of type `BookCase` consisting of models of type `Book`, you could write:

{% codeblock lang:javascript %}
var Book = PubSubItem.extend({
});

var Library = PubSubNode.extend({
    model: Book
});

var home = new BookCase([], {id: '/home/bookcase', connection: xmpp_connection});
home.fetch();
home.add({title: 'Developing Backbone.js Applications', author: 'Addy Osmani'});
home.save();
{% endcodeblock %}

which would

 * create a `BookCase` instance by passing its `id` and the Strophe `connection` in the options,
 * fetch it from the server,
 * add a book to the collection,
 * and persist to Pub-Sub.

 In your views you could be doing

{% codeblock lang:javascript %}

var BookCaseView = Backbone.View.extend({

    initialize: function () {
        this.on('add', this.addBook, this);
    },

    addBook: function (book) {
        var bview = new BookView({model: book});
        this.$el.append(bview.render().$el);
    },
    ....
});
{% endcodeblock %}

Now, if somebody else added a book to the bookcase, you would promptly receive a notification, the `add` event would be fired on your bookcase and the book's view would appear to your browser.

To visualize how this works, here's a quick demo of the [Todo.app](http://documentcloud.github.com/backbone/examples/todos/index.html) adapted to he XMPP storage. You can see the full code [here](http://github.com/ggozad/Backbone.xmpptodos).

{% vimeo 40685900 %}

Finally, if you do not want to receive push notifications on your models/collections or wish to craft your own events, you do not have to extend from `PubSubNode`, `PubSubItem`. In your own code, override `sync` in both the model and the collection and provide an instance of `PubSubStorage` as `node`on your collection. For instance:

{% codeblock lang:javascript %}
var MyCollection = Backbone.Collection.extend({
    sync: Backbone.xmppSync,
    model: MyModel,
    ...
});

var mycollection = new MyCollection();
mycollection.node = new PubSubStorage('mymodels', connection);
{% endcodeblock %}

and

{% codeblock lang:javascript %}
var MyModel = Backbone.Model.extend({
        sync: Backbone.xmppSync,
        ...
    });

var mymodel = new MyModel();
mycollection.add(mymodel);
{% endcodeblock %}

You can then subscribe to the XMPP notifications elsewhere, say for example in your collection view:

{% codeblock lang:javascript %}
connection.PubSub.on(
    'xmpp:pubsub:item-published:mymodels',
    this.itemPublished, this);
{% endcodeblock %}

There you go. Hope you will find this useful, will be looking forward to your contributions!