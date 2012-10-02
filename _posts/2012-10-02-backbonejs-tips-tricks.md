---
layout: post
tags: frontend
title: Backbone.js tips and tricks
---


## Call the same method on an array of objects

When you deal with BackboneJS you often have a view containing multiple 
subviews, sometimes you want to call a method on each subview. let us check 
it out with [_.invoke() function](http://underscorejs.org/#invoke):

{% highlight javascript %}

var view = {

    /**
     * Disabling this view will disable all subviews
     */
    disable: function() {
        // call disable() on each sub view
        _.invoke(this.subviews, 'disable')
    }

   // ... 
}
{% endhighlight %}


## Call a method only once (lazy connection)


If you have a database class, you need to establish the connection only 
once. Moreover, you want to do it only if you need it (lazy-loading). 
Let's see how this works with [_.once() function](http://underscorejs.org/#once):

{% highlight javascript %}
var db = {

    // will be called only once
    connect: _.once(function() {
        // connection code ..
    }),

    find: function(id) {
        // lazy connection
        this.connect()

        // find it
    },

    findall: function() {
        // lazy connection
        this.connect()

        // find all
    }
}
{% endhighlight %}

## Create a global event listener

In BackboneJS, each `Backbone.View`, `Backbone.Model`, `Backbone.Collection`, 
`Backbone.Router`n `Backbone.History` prototypes inherits from `Backbone.Events`, 
and it further means you have access to `on()`, `off()`, `trigger()` methods to manage your events.

The main problem with this approach is you have multiple listener instances. If you 
instanciate a `viewA` and another `viewB`, they do not have the same instance of 
`Backbone.Events`. That is to say, an event dispatched in `viewA` will not be received in `viewB`.

The workaround is to create a global event listener and inject it in each view:

{% highlight javascript %}
var eventListener = _.extend({}, Backbone.Events)

var viewA = new Backbone.View({ eventlistener: eventListener })
var viewB = new Backbone.View({ eventlistener: eventListener })
{% endhighlight %}


## Save expensive calls

For instance, you can use [_.debounce() function](http://underscorejs.org/#debounce), 
see [my previous post](http://jules.boussekeyt.org/2012/backbonejs-debounce.html) for more details on this. 


## Prevent double form submission

Double submission in a form is the worst thing that can happen to you. Fortunately with
the `_.debounce()` method it's easy to prevent it. What you have to do is set the third parameter
of the `debounce()` method to true. This way, the function is triggered when you click the submit 
button but it won't be triggered again in case the submit button is clicked the second time in quick 
succession (double submission).

{% highlight javascript %}
// prevent double click
$('button.my-button').on('click', _.debounce(function() {
    console.log('clicked')

    // code to handle form submition

}, 500, true)
{% endhighlight %}


## Don't mess with view.$el


Inside a Backbone view, the main dom element of the view is cached, so always use 
the `el` or `$el` property:

{% highlight javascript %}
var view = new Backbone.View()

// - get the jquery element of the view
view.$el // good
$(view.el) // wrong (bad performance)

// - get the dom element of the view
view.el // good
$(view.el)[0] // wrong (bad performance)
{% endhighlight %}


## Change the hash wby calling the router

Backbone.js applications state is managed by hashes (eg: #resource or #foo). Everytime the hash 
changes an action is called inside the `Backbone.Router`. This is usefull but in some case you may 
need to change the hash by calling the router. You can achieve it by passing true to `navigate()` 
method:

{% highlight javascript %}
var Router = Backbone.Router.extend({
    routes: {
        'foo': 'foo'
    },

    foo: function() {
        console.log('foo() called')
    }
})
var router = new Router()

Backbone.history.start()

dd// here the foo() action is called
window.location.hash = '#foo'
// => "foo() called"

// here the foo() action is not called
router.navigate('foo', true)
// => 

{% endhighlight %}


<div class="alert warning">
    Hey, I'm a <a href="/hire.html">Backbone.js freelance</a>, I'm specialized in Javascript applications, hire me!
</div>