---
layout: post
title: Ember Walkthrough
---

We are using ember-cli.

Make the first route:

    $ ember generate route nodes

This makes `app/routes/nodes.js` and a unit test also. Add a hook for the model data:


```javascript
export default Ember.Route.extend({
  model: function() {
    return this.store.find('node');
  },
});
```

You can also return JSON directly from a URL:

```javascript
model: function() {
    return ic.ajax({
        url: 'https://example.com/api/nodes',
        type: 'get'
    });
}
```

But the data store is so cool we want to use it instead.

* Get some test data in there: `ember g http-mock nodes`. This creates an Express server with an `api/nodes` endpoint. It's easy to paste JSON into the server file.

* Tell the data store where to find the data. `ember g adapter application`.The `namespace` property is prepended to the url used to fetch out models models:

```javascript
import DS from 'ember-data';

export default DS.RESTAdapter.extend({
  namespace: 'api'
});
```

* Display data via Handlebars template. In `templates/nodes.hbs`, do something like:

```
{% raw %}
{{#each node in model}}
  {{node.name}}
{{/each}}
{% endraw %}
```

The route is automatically resolved to this template because it has the same name as the route.


## Render multiple templates in one route ##
This is good so far, but we want to have an index route that shows our nodes data and also our field models.

* Make a template for the index route. `templates/index.js`:

```html
{% raw %}
<div>
    {{outlet nodes}}
    {{outlet fields}}
    {{outlet}}
</div>
{% endraw %}
```

This lets us render other templates inside our index template. Naming the outlets lets us specify them in the route object.

* Tell the index route what data to fetch and what templates to render. `routes/index.js` looks like this:


```javascript
import Ember from 'ember';
export default Ember.Route.extend({
  setupController: function(controller) {
  this.generateController('nodes');
      this.controllerFor('nodes').set('model', this.store.find('node'));

      this.generateController('fields');
      this.controllerFor('fields').set('model', this.store.find('field'));
  },
  renderTemplate: function (controller, model) {
      // index template
      this.render('index');

      // child templates
      this.render('nodes', {
        into: 'index',
        outlet: 'nodes',
      });
      this.render('fields', {
        into: 'index',
        outlet: 'fields',
      });
  },
});
```

Now Ember will look for a `nodes` and a `fields` template and render them inside the index template. We need to call `Route.generateController` and set the `model` property on the controller since we are not using a naming convention that tells Ember which models and templates to use. The `renderTemplate` hook lets us use other templates. We don't need to make the controller files for the node or field models--Ember will auto generate them.


## Show a list of models ##
This is something we are doing over and over again. This is the heart of front-end apps--showing people lists of things. Our two collection templates have the same code in them, how can we factor out the redundancy?

Use Ember components:

    $ ember g component model-list

This creates two files `templates/components/model-list.hbs` and `components/model-list.js`. The Handlebars template has the redundant stuff:

```html
{% raw %}
{{#each item in list}}
  <li>
    <a href="#">{{item.name}}</a>
    <a href="#">delete</a>
  </li>
{{/each}}
{% endraw %}
```

And the JS file subclasses `Ember.Component`. It has logic and stuff. Now in our collection templates, `fields.hbs` and `nodes.hbs`, we just have a single line: `{{model-list list=model}}`.

This creates an inplicit dependency--the component requires the models to have a `name` property. You could use Handlebars' [block form](http://emberjs.com/guides/components/wrapping-content-in-a-component/) to avoid this, though it is more complex.


## Interaction ##
[*Events in components*](http://emberjs.com/guides/components/sending-actions-from-components-to-your-application/)

Our buttons and links don't do anything. We need the [actions helper](http://emberjs.com/guides/templates/actions/). `{{action}}` emits a named event, so you can listen for it in the app code. The role of component actions is to translate a generic event like *click* into an event that has meaning in the application.

Define the list component so that it has two actions and passes the current model as a paramter:

```
{% raw %}
{{#each item in list}}
  <li>
    <a href="#" {{action "modelClick" item}}>{{item.name}}</a>
    <a href="#" {{action "modelDelete" item}}>delete</a>
  </li>
{{/each}}
{% endraw %}
```

In the component JS, pass the events up the hierarchy to the controller:

```javascript
export default Ember.Component.extend({
  tagName: 'ul',
  actions: {
    modelClick: function(model) {
      this.sendAction('modelClick', model);
    },
    modelDelete: function(model) {
      this.sendAction('modelDelete', model);
    }
  },
});
```

Map the component actions to specific action names in our `fields` template:

    {{model-list list=model modelClick="expandField" modelDelete="deleteField"}}

Finally handle the events in the fields controller:

```javascript
export default Ember.Controller.extend({
  actions: {
    expandField: function(field) {
      console.log("click field " + field.get('name'));
    },
    deleteField: function (field) {
      console.log("delete field " + field.get('name'));
    }
  }
});
```

The component event system gives us lots of layers which helps decouple things, though it's not as intuitive as a simpler system.


## We need a button ##
We need to replace that delete link with something better. We can make a reusable component for it.

    $ ember g component delete-button


## Model Relationships ##
Lets show the values that are related to each Field. They should be nested under the fields in the UI.

We can use the `{{render}}` helper. The first argument is the name of the template, the next argument is the model. Here calling field.values in the template returns an array of models.

`fields.hbs`:

```html
{% raw %}
<h2>Fields</h2>
<ul>
  {{#each field in model}}
    {{#model-item item=field modelClick="expandField" modelDelete="deleteField"}}
      {{render "values" field.values}}
    {{/model-item}}
  {{/each}}
</ul>
{% endraw %}
```

## Show details for a single model ##
We want to show all the attributes of a model in portion of the page. All the other page content will stay the same. This will happen when the user clicks a link. It should have a bookmarkable URL.

We can define a route like this:

```javascript
this.resource('index', {path: '/'}, function() {
    this.resource("node", {path: '/node/:node_id'});
});
```

When we navigate to `/node/1`, the `node` template will be rendered with the model with id 1, and appended to `{{outlet}}` in `index.hbs`. All the other page content in `index.hbs` will remain in place.

In `nodes.hbs`, we need to link to this route:

```html
{% raw %}
<ul>
  {{#each node in model}}
    <li>
      {{#link-to 'node' node}} {{node.name}} {{/link-to}}
    </li>
  {{/each}}
</ul>
{% endraw %}
```

`{%raw%}{{link-to}}{%endraw%}` takes the name of a route and a model.


## Relationships again ##
Implement ternary many-to-many relationship.

    $ ember g model triple

```javascript
export default DS.Model.extend({
  node: DS.attr('number'),
  field: DS.attr('number'),
  value: DS.attr('number'),
});
```

This maps the relationships between nodes, fields, and values.



