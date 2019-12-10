# Bacbbone.js Primer

## Introduction

Backbone.js gives structure to web applications by providing models with key-value binding and custom events, collections with a rich API of enumerable functions, views with declarative event handling, and connects it all to your existing API over a RESTful JSON interface.

### Immediately invoked functions

## Models

We can create models by calling `Backbone.Model({title: 'BackBone', Author: 'Ahmed'})`. However, to have a more customized model, we can extend the `Model` as follows `var ModelName = Backbone.Model.extend({})`

### Adding properties

  var NewModel = Backbone.Model.extend({
    prop1: 1,
    prop2: 'Two'
  });

*This is the constructor function for ModelName, and hence the use of Uppercase Naming*


### Class Properties

We can add static methods to the model by adding a second argument to the `Backbone.Model.extend()`:

  Backbone.Model.extend({},
    {
      summary: function() {
        return 'Some text';
      }
    }
  );

## Notes

### `instanceof` keyword

`console.log(a instanceof A);` returns a boolean of whether the instance `a` is of type `A` or not.
  

