---
layout: post
author: dagingaa
title: Understanding template compiling in AngularJS
category: Technology
---
One of the first things that stumped me when I first got into AngularJS was how
template compiling worked. In other frameworks, I was used to templates being a
dumb string, which had some special characters that would output data from an
object I put into it. However, this meant that we were working on pure strings.
The input was a string, and the output was a string. You just dumped everything
into the DOM where you wanted it with `.innerHTML()`.

I quickly realised that AngularJS must work in another way. It had two-way data
bindings, working in perfect sync to keep my view updated with the latest copy
of my model. It automatically registered event listeners with `ng-click`, and
could even sync data from an input field into my model, like magic!

But, it's not magic. The concepts of turning a dumb template into a dynamic view
with data bindings and event listeners is actually quite simple, and once you
grasp how that works, you will also understand your application a lot better.

Let's look at the most basic example there is. This is our application:

{% highlight html linenos %}
<div ng-app>
    <label>Name:</label>
    <input type="text" ng-model="yourName" placeholder="Enter a name here">
    <h1>Hello { {yourName} }!</h1>
</div>
{% endhighlight %}

As you can see, this is the very same basic application that Angular has on
their frontpage. There is no custom code involved, just the Angular library.

## Inversion of Control

So, the first principle you need to get to know is [Inversion of
Control](https://en.wikipedia.org/wiki/Inversion_of_control). Instead of
manually bootstrapping your application, Angular assumes that you have a few
basic things in place, and that you follow a couple of rules (it has to be said
that a colleague of mine wrote a [quite popular blog
post](http://comoyo.github.io/blog/2013/02/06/the-inverse-of-ioc-is-control/)
about why IoC can be an anti-pattern).

For example, Angular assumes that you have an element attribute called `ng-app`
present on the element which will be the root node of your Angular application.
This allows Angular to assume full control of that element and all its child
nodes, and makes it possible for Angular to live side-by-side with other
JavaScript frameworks. So, when Angular bootstraps your application for you, it
will start by going through your DOM and looking for that attribute, called a
*directive* in Angular-lingo. Note that the `ng-app` directive can live on any
DOM node, including the `<html>` and `<body>` tags.

Angular also assumes that no other framework will touch anything inside this
root node, and doing so can lead to unexpected behaviour and potential breakage
of the two-way data bindings and event listeners.

## A closer look at the template
{% highlight html linenos %}
<label>Name:</label>
<input type="text" ng-model="yourName" placeholder="Enter a name here">
<h1>Hello { {yourName} }!</h1>
{% endhighlight %}
<small>Jekyll's Markdown parser doesn't like { }, so there shouldn't be a space
between them.</small>

This is the template. Everything that lives inside the root node is considered
to be a dumb template, ready for compiling. A template, together with the model
(`$scope`) and controller, is what makes up the dynamic view that the user can
see and interact with in the browser. In this case, there is no controller or
explicit model. Instead, Angular will create these bits for us behind the
scenes.

Every AngularJS application has a `$rootScope`. This is responsible for holding
references to all child scope below it, and can itself contain data shared by
all `$scope`s. In this case, our template will be bound to this `$rootScope` in
lack of a controller that sets up its own child `$scope`.

Angular is also forgiving. As we can see in the template, we have bound a model
to our input field, which will register an event listener for the keydown event
on our input element, and automatically store everything we write into
`$rootScope.yourName`. We haven't declared this variable, and we don't need to.
A scope variable can initially be undefined, just like a regular JavaScript
Object.

We also have an
[$interpolate](http://docs.angularjs.org/api/ng/service/$interpolate) directive,
denoted by the squiggly brackets: { { yourName } }. This will create a
[$watcher](http://docs.angularjs.org/api/ng/type/$rootScope.Scope#$watch) that
will listen for changes on the model and update the view if it changes. More on
this in a later blog post.

Now, we just need to turn these dumb DOMElements into a dynamic view.

## Compiling the template
After Angular has found the `ng-app` directive, it will start up, create a new
`$rootScope`, and start compiling the child DOMElements of the root node (the
template above).

What happens now is a two step process, compiling and linking. For a more in
depth introduction, please refer to the [AngularJS documentation on the HTML
Compiler](http://docs.angularjs.org/guide/compiler).

First off, Angular's
[$compile](http://docs.angularjs.org/api/ng/service/$compile) function takes our
DOMElements as input. This is different from other frameworks, as Angular's
$compile function traverses the DOM using browser API's, instead of doing string
substitution. Should you still need to pass in strings, you can first transform it into
DOMElements using the `angular.element` function.

The $compile function traverses the DOM and looks for directives. For each
directive it finds, it adds it to a list of directives. Once the entire DOM has
been traversed, it will sort that list of directives by their priority. Then,
each directive's own compile function are executed, giving each directive the
chance to modify the DOM itself. Each compile function returns a *linking*
function, which is then composed into a "combined" linking function and
returned.

Then, Angular executes the returned linking function, making sure to pass in the scope we
want to bind to in the process. This will run all the children linking functions
and binding the same scope, or, depending on the directive, create new child
scopes. Once all linking functions has been run, the combined linking function
returns a set of DOMElements, complete with data bindings and event listeners,
which Angular can append to the parent node.

The pseudo code for the process above can be seen as such:
{% highlight javascript linenos %}
var $compile = ...; // injected into your code
var $rootScope = ...; // injected
var parent = ...; // DOM element where the compiled template can be appended
var template = ...; // Our template DOMElements from above

var linkFn = $compile(template);

var element = linkFn(scope);

parent.appendChild(element);
{% endhighlight %}

## Summary
And that's about it! As we can see, AngularJS isn't all that different from
other frameworks, it just sets a few conventions and assumes a couple of things
based on those conventions, so you can spend less time writing bootstrapping
code, and more time writing your application. However, this is just part of the
magic in AngularJS, stay tuned for an update on how $watch and $digest work to
keep your views fresh.

