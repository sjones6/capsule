# Capsule

(Name subject to change once I release on NPM)

Better state management for JavaScript objects.

## Installation

This package is not on NPM yet, so you'll need to clone the repo and install it manually. This can be done easily by requiring the main capsule file.

Clone the repository:
```
git clone https://github.com/sjones6/capsule.git
```

Require Capsule into your project:
```javascript
const Capsule = require('./path/to/capsule');
```

There are no explicit dependencies, except development dependencies. If you want to contribute, install the development dependencies:

with NPM: `npm install`

or 

with Yarn: `yarn`.

## Getting Started

Capsule works by leveraging the "privacy" of closures in Javascript functions, and especially constructor functions.

In order for Capsule to provide help with an object's private state, you'll need create a new instance of Capsule inside of your constructor function. Simply never set the Capsule instance on the object constructed and it will not be accessible
outside of that closure.

Effectively, this means that the capsule instance is available only to the object created by the constructor.

```javascript
const Capsule = require('capsule');

// Here's your constructor function
let Person = function(name) {
 
  // Capsule takes two params, and only one is required
  // 1.) Required: An object with property declarations
  // 2.) Optional: options to loosen strictness
  store = new Capsule({
    props: {
        
        // Declare your property names here
        // and their types.
        name: String,
        emails: Array,
        phone: Number,
        hasChildren: Boolean,

        // You can use this alternative syntax
        // to declare an initial/default value 
        // along with the type declaration
        location: {
            type: Object,
            default: {
                lat: "",
                long: ""
            }
        },

        // Type checking of custom
        // classes is supported
        spouse: Person,

        // The type can be set to null to
        // disable type checking.
        favoriteSports: null
    },

    // Computed properties are not yet supported
    // but are in development
    computed: {
    }
  }, {

    // Whether to enforce strict types
    // during setting. Defaults to true
    strictTypes: true,

    // Whether to freeze after initialization;
    // If frozen, store's properties can be
    // manipulated but no properties added.
    // Defaults to true.
    freeze: true
  });

  store.name = name;
    
  this.greet = function() {
    return `Hi, ${store.name}!`;
  };
  this.addEmail = function(email) {
      store.emails.push(email);
  }
  this.setLocation = function(location) {
      store.location = location;
  }
  this.getLocation = function() {
      return store.location;
  }
    
}

let jane = new Person('jane');
let bob = new Person('bob');

jane.greet(); // Hi, jane

bob.addEmail('bob@email.com');
bob.emails; // undefined

jane.setLocation('nowhere'); // throws type error since location expects an object not string
```

## Supported Types

1. String
2. Number
3. Boolean
4. Object
5. Function
6. Classes (including internal and user defined classes)
7. Any type

These can be declared in the following manner:

```javascript
let capsule = new Capsule({
  props: {
    string: String,
    number: Number,
    boolean: Boolean,
    object: Object,
    function: Function,
    internalClass: Date,
    customClass: MyCustomClass,
    anyType: null
  }
});
```

Type checking can also be disabled by passing `{strictTypes: false}` in the options parameter.

## Subscribing to Updates

You can subscribe to property updates:

```javascript
capsule.subscribe('propName', (newValue, oldValue) => {
    // handle update
});

// Removes subscription
capsule.unsubscribe('propName'); 
```

Example:
```javascript
let capsule = new Capsule({
  props: {
    name: {
      type: String,
      default: ''
    }
  }
});

capsule.subscribe('name', (newName, oldName) => {
    // update logic
});

capsule.name = 'New name'; // fires subscribe handler
```

Only one subscribe handler is supported for each property. Adding a new subscribe handler will overwrite the previous one.

## Yeah, but isn't it slower than plain JavaScript objects?

Yes, of course. Capsule adds some overhead to getting and setting properties, but the payoff is better state-management.

You will have to decide if Capsule's overhead is an acceptable cost. In some applications, especially applications where state-management is not that complicated, Capsule may not be necessary. However, in complex applications, it becomes increasingly critical to manage state in ways that are easy to reason about.

If you're already doing a lot of type checking and enforcing setter methods in your objects, then Capsule takes a lot of this pain away so that you can focus on creating and not on boilerplate.

I feel that the time spent debugging mis-managed state more than makes up for the performance difference. You can decide if Capsule is right for your application.

However, if you want some raw numbers about how Capsule performs:

Raw Object Properties (no accessor methods): ~40-42k operations/ms

Object with accessor methods: ~21-22k operations/ms

Capsule: ~18-19k operations/ms

To profile your machine, install Capsule and run `npm run capsule-profile` (must have Node and npm installed).

## In Development

Computed properties.

## Running Tests

`npm run test`

Powered by Mocha.

