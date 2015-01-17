---
layout: post
title: Model Relationships
---

The trials of client side model relationships.

Problem: How to do ORM on the client side? This is a well-trod path on the server. That's what Rails apps are. This is very new on the client side. Web browsers now have access to a simple database, so ORM is possible in the browser, though it requires more application code because the DB is less sophisticated, and the frameworks are not mature.

Requirements:
3 types of models: Nodes, Fields, Values.

* Nodes have many Fields
* A Node + Field pair is related to a set of Value objects.
* A Field + Value pair is related to a set of Nodes.
* Values can have sub-values (hierarchy)
* Values have one Field. If a value with the same name appears under different Fields, those are different values.

Normally you would have Nodes (JSON objects) with fields and values as metadata. That's not adequate because Values have their own metadata (they can be related to other Values).

This is a schema for modeling arbitrary data. It is the same idea used for RDF, where it is called a triple store or graph database.

## Backbone ##
Two simple solutions in Backbone: join table or references stored on the model. See this [video](https://www.youtube.com/watch?v=UAl_N62gKmM#t=2049).

Complexity arises from maintaining synchronicity between our domain layer and our database. Good for friendship example, harder with ternary many-to-many relationships.

## Ember ##
"Modeling relationships between records is easily the most difficult feature we've added to Ember Data."

"...everyone who tries to implement their own domain-specific relationships quickly ends up with their own ad hoc mini-framework."

- [ember blog](http://emberjs.com/blog/2014/03/18/the-road-to-ember-data-1-0.html)

Video: [Ember Data: Tips, Tricks, & Lessons Learned](https://www.youtube.com/watch?v=HL2bMjndviE)

Video: [Ember Data and the Way Forward](https://www.youtube.com/watch?v=D3mC14NfJwc)

Get your data by querying the *data store*, an in-memory cache of your models. The data store does not return the model object, it returns a *promise* for your model. All methods for getting models are async via promises.

Can you further generalize Ember data store (a model cache) into an NPM module / generic JS component?

**DataBoundPromise**
Allows you to observe properties on the Promise in certain contexts. Relies on Ember infrastructure.

See [here](https://www.youtube.com/watch?v=D3mC14NfJwc#t=1360). Ember data originally provided an API with both synchronous and async methods. You would use the sync API if you knew that the model was already loaded in memory. They changed to all async because it is better.

All getting of related models is done with promises. This is how you would get the name of the first friend of this user:

```javascript
user.get('friends').then( function(friends){
	friends.objectAt(0).then( function(firstFriend){
		firstFriend.get('name');
	});
});
```

All models are accessed through the `store` object, an identity map.

Better to put fetch methods on models or on another object? Does it matter?

	myModel = new MyModel({ id: 1});
	myModel.fetch();

vs:

	// models have data store as a property
	myModel = this.dataStore.find({id: 1}).then( function(myModel) {
		return myModel;
	)};

In the first example it is possible to create a model isntance that does not have its data loaded.


### Adapter ###
Responsible for translating requests for models into calls to the server.

I need a model:
```
app code --> data store --> adapter --> server or something
```

This allows us to program the adapter to an interface that the datastore is expecting. Now we can swap out our persistance layer by just changing the adapter.

A specific adapter extends `DS.Adapter`, an abstract class.

The adapter is set application-wide as a property: `MyApp.ApplicationAdapter = MyApp.CustomAdapter`.

Model-specific adapters are created by assigning the adapter to a specifically named property on the application object:

	var MyPostAdapter = DS.Adapter.extend({
	  // ...Post-specific adapter code goes here
	  });

	  App.PostAdapter = MyPostAdapter;

We can't put this as a property on the model because the models talk to the datastore? They don't know about adapters?

In Backbone the adapter is a static function, `.sync` on the Backbone object.


## Ampersand ##
[ampersand-registry](https://github.com/AmpersandJS/ampersand-registry/blob/master/ampersand-registry.js)

This is different because it is much simpler. No async, models are still fetched directly from the `.sync` method.

Part of the value of frameworks in JS comes from the fact that there is no such thing as an interface. There is no compile time checking. That makes it hard for generic modules to interoperate.



