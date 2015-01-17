---
layout: post
title: Ember Notes
---

# Ember Notes #
`Ember` is aliased as `Em`.

## Overview ##

* the user enters a URL, and the Router regonizes it and passes it to a route object.

* The route object instantiates a Controller and a Model.

* The view layer (a Handlebars template) has access to the state in the Controller. The Controller contains its own state, plus proxies the Model's state.

In the above steps, if the controller or template don't exist, Ember will auto generate one for you. This takes some of the drudgery out of having so many abstractions. If you don't need the sophistication, you don't have to write the code.

This is how you start the application:

	window.app = Ember.Application.create();

The app is namespaced as a window global, like in traditional JS, but this doc seems old. I think ember-cli uses a different system? Or hides the namespacing via ES6 modules. Creating this instance renders the `application` template.


## Controllers ##
Controllers have properties that will not be persisted, like state that is specific to display.

A template gets data from a controller.

A controller is used for each route. In a Route object, you configure the controller via hooks.

When transitioning between routes, the router collects all the models needed for that route. The `model` hook can return a promise instead of a model instance, making it async.

Model doesn't know about controller, controller doesn't know about template.

	template --> controller --> model

Dependencies between controllers are handled via configuration:

	App.CommentsController = Ember.ArrayController.extend({
	  needs: "post"
	});

The `needs` property names another controller. This is [dependecy injection](http://emberjs.com/guides/understanding-ember/dependency-injection-and-service-lookup/)

? I never quite got DI because I don't understand why it is better than simply putting dependencies in the constructor.

Controllers are *singletons*.

## DI ##
DI makes me barf. Why do we need this instead of requiring dependencies in a constructor? There is a magic soup of flags and conventions for resolving dependencies.

When the app starts, it creates a single instance of `Ember.Container`. This object manages factories and their dependencies.

Factories use a naming convention. An application view would be named `view:application`.

The container is not part of a public API.

## Views ##
The view translates primitive browser events into events that have meaning to your application. They can also be used to create re-usable components.
A lot of the display logic is in the Handlebars templates, so views have a smaller role than in other frameworks.

Views know which template to use because of `templateName` property. Templates have access to the view's state via `{{view}}` helper. Append a view to an elmt with `view.appendTo('#container')`.

The view's context is its controller. It can be set as a property: `view.controller` or `view.context`, or, if those are not defined, it uses it's parent's context. [See docs](http://emberjs.com/api/classes/Ember.View.html#toc_view-context).


## View vs Component ##
I want to make re-usable parts for the app, which one do I use? Apparently views are becoming obsolete. Components now do the same stuff. [stackoverflow](http://stackoverflow.com/questions/18593424/views-vs-components-in-ember-js)


## Templates ##
Templates do most of the view-like work in Ember.

The content in the `{{outlet}}` helper changes based on the URL/router.


## Components ##
When used inside a template, the event gets sent to the controller then the routes, bubbling up the hierarchy. If nothing handles the event, an error is thrown. When used inside a component, though, the action is only sent to the component instance. Components need to explicitly pass events up the hierarchy by using the `sendAction()` method. When using a component, set its primary action like this: `{{my-button action="showUser"}}`.

Pass the event plus parameters: `this.sendAction('action', this.get(param));`
Use the component like this: `{{confirm-button title="Delete" action="deleteTodo" param=todo}}`. That sets `component.param` to the todo model instance.

If someone using the component has not specified an action, then calling `sendAction()` has no effect.

Pass arguments like `{{action "action_name" argument}}`. Arguments come from the same context as the template.

They listen to click events by default. Use other events: `{{action "select" post on="mouseUp"}}`.

Use `bubble=false` to stop event bubbling within DOM nodes.


## vs. Backbone ##
Both Ember and Ampersand separate the session state from the model's persisted state. In Ember the session state is handled by the controller, in Amp it is kept in a separate property on the model. This is nice.









## Router ##
Maps URLs to Route objects.

## Routes ##
* Which templates do we render?
* Where do we insert the templates?
* Update or set up a Controller
* Fetch models

### Route Hooks ###
* `setupController(controller, model)`
Called with the model supplied by `model` hook. The default, inherited method sets the `model` property of the controller to the `model` parameter.

* `model(params, transition, queryParams)`
Returns the model used by this controller. It's ok to return a promise for the model.


## Templates ##
* Use `{{render template_name model}}` to render a template inside of another template. It will be setup with the model's controller as context.





