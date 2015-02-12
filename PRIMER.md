WORK IN PROGRESS

# Polymer 0.8 Primer

Table of Contents:

* [Feature layering](#feature-layering)
* [Polymer micro features](#polymer-micro)
* [Polymer mini features](#polymer-mini)
* [Polymer standard features](#polymer-standard)
* [Migration notes](#migration-notes)

<a name="feature-layering"></a>
# Feature layering

Polymer 0.8 is currently layered into 3 sets of features provided as 3 discrete HTML imports, such that an individual element developer can depend on a version of Polymer whose feature set matches their tastes/needs.  For authors who opt out of the more opinionated local DOM or data-binding features, their element's dependencies would not be payload- or runtime-burdened by these higher-level features, to the extent that a user didn't depend on other elements using those features on that page.  That said, all features are designed to have low runtime cost when unused by a given element.

Higher layers depend on lower layers, and elements requiring lower layers will actually be imbued with features of the highest-level version of Polymer used on the page (those elements would simply not use/take advantage of those features).  This provides a good tradeoff between element authors being able to avoid direct dependencies on unused features when their element is used standalone, while also allowing end users to mix-and-match elements created with different layers on the same page.

Below is a description of the current Polymer layers and included features, followed by individual feature guides.

## polymer-micro.html
Bare-minum Custom Element sugaring

| Feature | Usage
|---------|-------
| [Custom element constructor](#element-constructor) | Polymer.Class({ … });
| [Custom element registration](#register-element) | Polymer({ is: ‘...’,  … }};
| [Bespoke constructor support](#bespoke-constructor) | constructor: function() { … }
| [Basic lifecycle callbacks](#basic-callbacks) | created, attached, detached, attributeChanged
| [Native HTML element extension](#type-extension) | extends: ‘…’
| [Publish API](#published-api) | published: { … }
| [Attribute deserialization to property](#attribute-deserialization) | published: { \<property>: \<Type> }
| [Set boolean host attributes](#host-attributes) | hostAttributes: [ … ]
| [Module registry](#module-registry) | modularize, using
| [Prototype Mixins](#prototype-mixins) | mixins: [ … ]

## polymer-mini.html
Custom Elements with Templates stamped into "local DOM"

| Feature | Usage
|---------|-------
| [Template stamping into local DOM](#template-stamping) | \<dom-module>\<template>...\</template>\</dom-module>
| [DOM (re-)distribution](#dom-distribution) | \<content>
| [Local & light tree API](#dom-api)  | localDom, lightDom
| [Top-down callback to configure defaults](#configure-method) | configure: function() { … }
| [Bottom-up callback after configuration](#ready-method) | ready: function() { … }

## polymer.html (standard)
Custom elements with declarative data binding, events, and property nofication

| Feature | Usage
|---------|-------
| [Local node marshalling](#node-marshalling) | this.$.\<id>
| [Event listener setup](#event-listeners)| listeners: { ‘\<node>.\<event>’: ‘function’, ... }
| [Annotated event listener setup](#annotated-listeners) | \<element on-[event]=”function”>
| [Key listener setup](#key-listeners) | keyPresses: { '\<char>' \| \<code>: ‘function’, … }
| [Property change callbacks](#change-callbacks) | bind: { \<property>: ‘function’ }
| [Declarative property binding](#property-binding) | \<element prop=”{{property\|path}}”>
| [Property change notification](#property-notification) | published: { \<prop>: { notify: true } }
| [Binding to structured data](#path-binding) | \<element prop=”{{obj.sub.path}}”>
| [Path change notification](#set-path) | setPathValue(\<path>, \<value>)
| [Declarative attribute binding](#attribute-binding) | \<element attr$=”{{property\|path}}”>
| [Reflecting properties to attributes](#attribute-reflection) | published: \<prop>: { reflect: true } }
| [Computed properties](#computed-properties) | compute: { \<property>: ‘function(\<property>)’ }
| [Read-only properties](#read-only) |  published: { \<prop>: { readOnly: true } }
| [Utility functions](#utility-functions) | toggleClass, toggleAttribute, fire, async, …
| [Attribute-based layout](#layout-html) | layout.html (layout horizontal flex ...)

<a name="polymer-micro"></a>
# Polymer Micro Features

<a name="element-constructor"></a>
## Custom Element Constructor

The most basic Polymer API is `Polymer.Class({...})`, which takes an object expressing the prototype of your custom element, chains it to Polymer's `Base` prototype (which provides value-add features described below), and returns a constructor that can be passed to `document.registerElement()` to register your element with the HTML parser, and after which can be used to instantiate new instances of your element via code.

The only requirement for the prototype passed to `Polymer.Class` is that `is` property specifies the HTML tag name the element will be registered as.

Example:

```js
var MyElement = Polymer.Class({

  is: 'my-element',

  // See below for lifecycle callbacks
  created: function() {
    this.innerHTML = 'My element!';
  }

});

document.registerElement('my-element', MyElement);

// Equivalent:
var el1 = new MyElement();
var el2 = document.createElement('my-element');
```

`Polymer.Class` is designed to provide similar ergonomics to a speculative future where an ES6 class may be defined and provided to `document.registerElement` to achieve the same effect.

<a name="register-element"></a>
## Custom Element Registration

Because the vast majority of users will always want to register the custom element prototype generated by Polymer, Polymer provides a `Polymer({ ... })` function that wraps calling `Polymer.Class` and `document.registerElement`.

Example:

```js
MyElement = Polymer({

  is: 'my-element',

  // See below for lifecycle callbacks
  created: function() {
    this.innerHTML = 'My element!';
  }

});

var el1 = new MyElement();
var el2 = document.createElement('my-element');
```

<a name="bespoke-constructor"></a>
## Bespoke constructor support

While the standard `Polymer.Class()` and `Polymer()` functions return a basic constructor that can be used to instance the custom element, Polymer also supports providing a bespoke `constructor` function on the prototype that can, for example, accept arguments to configure the element.  In that case, the constructor should generally call `document.createElement(this.is)` to construct the element and return the element instance after constructing it.

Example:

```js
MyElement = Polymer({

  is: 'my-element',

  constructor: function(foo, bar) {
    var el = document.createElement(this.is);
    el.foo = foo;
    el.configureWithBar(bar);
    return el;
  },

  configureWithBar: function(bar) {
    ...
  }

});

var el = new MyElement(42, 'octopus');
```

<a name="type-extension"></a>
## Native HTML element extension

Polymer 0.8 currently only supports extending native HTML elements (e.g. `input`, `button`, etc., as opposed to [extending other custom elements](#todo-inheritance)).  To extend a native HTML element, set the `extends` property to the tag name of the element to extend.


Example:

```js
MyInput = Polymer({

  is: 'my-input',

  extends: 'input',

  created: function() {
    this.style.border = '1px solid red';
  }

});

var el1 = new MyInput();
console.log(el1 instanceof HTMLInputElement); // true

var el2 = document.createElement('input', 'my-input');
console.log(el2 instanceof HTMLInputElement); // true
```

<a name="basic-callbacks"></a>
## Basic lifecycle callbacks

Polymer's Base prototype implements the standard Custom Element lifecycle callbacks to perform tasks necessary for Polymer's built-in features.  The hooks in turn call shorter-named lifecycle methods on your prototype.

- `created` instead of `createdCallback`
- `attached` instead of `attachedCallback`
- `detached` instead of `detachedCallback`
- `attributeChanged` instead of `attributeChangedCallback`

You can always fallback to using the low-level methods if you wish (in other words, you could simply implement `createdCallback` in your prototype).

Example:

```js
MyElement = Polymer({

  is: 'my-element',

  created: function() {
    console.log(this.localName + '#' + this.id + ' was created');
  },

  attached: function() {
    console.log(this.localName + '#' + this.id + ' was attached');
  },

  detached: function() {
    console.log(this.localName + '#' + this.id + ' was detached');
  },

  attributeChanged: function(name, type) {
    console.log(this.localName + '#' + this.id + ' attribute ' + name +
      ' was changed to ' + this.getAttribute(name));
  }

});
```

`Polymer.Base` also implements `registerCallback`, which will be called by `Polymer()` to allow `Polymer.Base` to supply a layering system for Polymer abstractions.

See the [section on configuring elements](#configuring-elements) for a more in-depth description of the practical uses of each callback.


<a name="published-api"></a>
## Published API

Placing an object-valued `published` property on your prototype allows you to define metadata regarding your Custom Element's API, which can then be accessed by an API for use by other Polymer features.

By itself, the `published` feature **doesn't do anything**. It only provides API for asking questions about these special properties (see featues below for details).

Example:

```js
Polymer({

  is: 'x-custom',

  published: {
    user: String,
    isHappy: Boolean,
    count: {
      type: Number,
      readOnly: true,
      notify: true
    }
  },

  created: function() {
    this.innerHTML = 'Hello World, I am a <b>Custom Element!</b>';
  }

});
```

Remember that the fields assigned to `count`, such as `readOnly` and `notify` don't do anything by themselves, it requires other features to give them life, and may depend on which layer of Polymer is in use.

<a name="attribute-deserialization"></a>
## Attribute deserialization

If an attribute matches a property listed in the `published` object, the attribute value will be assigned to a property of the same name on the element instance.  Attribute values (always strings) will be automatically converted to the published type when assigned to the property.  If no other `published` options are specified for a property, the type (specified using the type constructor, e.g. `Object`, `String`, etc.) can be set directly as the value of the property in the published object; otherwise it should be provided as the value to the `type` key in the `published` configuration object.

The type system includes support for Object values expressed as JSON, or Date objects expressed as any Date-parsable string representation. Boolean properties set based on the existence of the attribute: if the attribute exists at all, its value is true, regardless of its string-value (and the value is only false if the attribute does not exist).

Example:

```html
<script>

  Polymer({

    is: 'x-custom',

    published: {
      user: String,
      manager: {
        type: Boolean,
        notify: true
      }
    },

    created: function() {
      // render
      this.innerHTML = 'Hello World, my user is ' + (this.user || 'nobody') + '.\n' +
        'This user is ' + (this.manager ? '' : 'not') + ' a manager.';
    }

  });

</script>

<x-custom user="Scott" manager></x-custom>
<!--
<x-custom>'s innerHTML becomes:
Hello World, my user is Scott.
This user is a manager.
-->
```

**Warning:** Currently only lower-case published properties are supported.  Camel-case property support will be added in this sprint.

<a name="host-attributes"></a>
## Boolean host attributes

A list of attribute names to be applied to instances of the custom element can be provided as space-separated strings in the `hostAttributes` property.  These will simply be set on the element during creation.  This is intended for "boolean" attributes only, such as common layout attributes used by the [layout.html](layout-html) CSS; attribute values cannot be supplied at this time.

Example:

```html
<script>

  Polymer({

    is: 'x-custom',

    hostAttribute: 'layout horizontal fit'

  });

</script>
```

After creation:

```html
<x-custom layout horizontal fit></x-custom>
```

<a name="module-registry"></a>
## Module registry

Polymer provides and internally uses a JavaScript "module registry" to organize library code defined outside the context of a custom element prototype, and may be used to organize user code when convenient as well.  The registry is responsible for storing and retrieving JS modules by name.  As this facility does not provide dependency loading, it is the responsibility of the user to HTMLImport files containing any dependent modules before use.

Modules are registered using the `modulate` global function, passing a name to register and a factory function that returns the module:

`fun-support.html`

```js
<script>
modulate('FunSupport', function() {

  return {
    makeElementFun: function(el) {
      el.style.border = 'border: 20px dotted fuchsia;';
    }
  };

});
</script>
```

A list of module dependencies can be specified as the second parameter, which will be provided to the factory function:

`fun-support.html`

```js
<script>
modulate('FunSupport', ['Squid', 'Octopus'], function(squid, octopus) {

  // use squid & octopus
  return ...;

});
</script>
```

Modules are requested using the `using` global function, passing a list of dependencies:

`my-element.html`

```html
<!-- Load module dependency -->
<link rel="import" href="fun-support.html">

<script>
// Use module dependency
using(['FunSupport', ...], function(funSupport, ...) {

  MyElement = Polymer({

    is: 'my-element',

    created: function() {
      funSupport.makeElementFun(this);
    }

  });

});
</script>
```


<a name="prototype-mixins"></a>
## Prototype mixins

Polymer will "mixin" objects specified in a `mixin` array into the prototype.  This can be useful for adding common code between multiple elements.

The current mixin feature in 0.8 is basic; it simply loops over properties in the provided object and adds property descriptors for those on the prototype (such that `set`/`get` accessors are copied in addition to properties and functions).  Note that there is currently no support for publishing properties or hooking lifecycle callbacks directly via mixins.  The general pattern is for the mixin to supply functions to be called by the target element as part of its usage contract (and should be documented as such).  These limitations will likely be revisited in the future.

The [module registry](#module-registry) should generally be used for registering mixins.  Mixins registered with the Polymer module registry may be referred to by String name without needing to expliitly request the module via `using`.  Otherwise, values in the `mixin` array should be an Object reference (generally retrieved via `using`).

Example: `fun-mixin.html`

```js
modulate('FunMixin', function() {

  return {
    funCreatedCallback: function() {
      this.makeElementFun();
    },

    makeElementFun: function() {
      this.style.border = 'border: 20px dotted fuchsia;';
    }
  };

});
```

Example: `my-element.html`

```html
<link rel="import" href="fun-mixin.html">

<script>
Polymer({

  is: 'my-element',

  mixins: ['FunMixin'],

  created: function() {
    this.funCreatedCallback();
  }

});
</script>
```

<a name="polymer-mini"></a>
# Polymer Mini Layer

<a name="template-stamping"></a>
## Template stamping into local DOM

We call the dom which an element is in charge of creating an managing its `local DOM`. This is distinct from the element's children which are sometimes called its `light DOM` for clarity.

To specify dom to use for an element's local DOM, use the `<dom-module>` element.
Give the `<dom-module>` an `id` attribute that matches its element's
`is` property and put a `<template>` inside the `<dom-module>`.
Polymer will automatically stamp this template into the element's local DOM.

Example:

```html
<dom-module id="x-foo">
  <template>I am x-foo!</template>
</dom-module>

<script>
  Polymer({
    is: 'x-foo'
  });
</script>
```

We say that an element definition has an imperative and declarative portion. The imperative
portion is the call to `Polymer({...})`, and the declarative portion is the `<dom-module>`
element. The imperative and declarative portions of an element's definition may be placed
in the same html file or in separate files.

**NOTE:** Defining an element in the main html document is not currently supported.

**NOTE:** Polymer also currently supports locating the element's template at the node previous
to the element's script element; however, this may be deprecated.

<a name="dom-distribution"></a>
## DOM (re-)distribution

To support composition of an element's light DOM with its local DOM, Polymer supports the `<content>` element. The `<content>` element provides an insertion point at which an element's light DOM is combined with its local DOM. The `<content>` element supports a `select` attribute which filters nodes via a simple selector.

Polymer supports multiple local DOM implementations. On browsers that support ShadowDOM, ShadowDOM may be used to create local DOM. On other supported browsers, Polymer provides local DOM via a custom implementation called ShadyDOM which is inspired by and compatible with ShadowDOM.

Example:

```html
<template>
  <header>Local dom header followed by distributed dom.</header>
  <content select=".content"></content>
  <footer>Footer after distributed dom.</footer>
</template>
```
<a name="dom-api"></a>
## DOM API

Polymer provides custom API for manipulating dom such that local DOM and light DOM trees are properly maintained. An element has a `lightDOM` property and a `localDOM` each of which are used to manipulate their respective dom trees. The following methods are provided:

  * appendChild(node)
  * insertBefore(node, ref_node)
  * removeChild(node)
  * querySelector/querySelectorAll(selector)
  * children()
  * elementParent(node)

Example:

```html
  var toLight = document.createElement('div');
  this.lightDOM.appendChild(toLight);

  var toLocal = document.createElement('div');
  this.localDom.insertBefore(toLocal, this.localDom.children()[0]);

  var allSpans = this.localDOM.querySelectorAll('span');
```

For manipulating dom in elements that themselves do not have local dom, the above api's support an extra argument which is the container `node` in which the operation should be performed.

Example:

```html
<template>
  <div id="container">
     <div id="first"></div>
     <content></content>
  </div>
</template>

...

var insert = document.createElement('div');
this.localDOM.insertBefore(insert, this.$.first, this.$.container);

```

**NOTE:** It's only strictly necessary to use `lightDOM/localDOM` when performing dom manipulation on elements whose composed dom and local dom is distinct. This includes elements with local dom and elements that are parents of insertion points. It is recommended, however, that `lightDOM/localDOM` be used whenever manipulating element dom.

When multiple dom operations need to occur at once time, it's more efficient to batch these operations together. A batching mechanism is provided to support this use case.

Example:

```html
this.localDOM.batch(function() {
  for (var i=0; i<10; i++) {
    this.localDOM.appendChild(document.createElement('div'));
  }
});

```

To provide a `local` view of the dom tree from a perspective independent of a custom element, polymer provides the `Polymer.dom` object. This object supports `querySelector/querySelectorAll` for example and the following guarantees that elements inside local DOM will not be seen.

Example:

```html
  Polymer.dom.querySelector('#myId');
```

Sometimes it's necessary to access the elements which have been distributed to a given `<content>` insertion point or to know to which `<content>` a given node has been distributed. The `distributedNodes` and `destinationInsertionPoints` respectively provide this information.

Example:

```html
  <x-foo>
    <div></div>
  </x-foo>

  // x-foo's template
  <template>
    <content></content>
  </template>

  ...

  var div = xFoo.lightDom.querySelector('div');
  var content = xFoo.localDom.querySelector('content');
  var distributed = xFoo.localDom.distributedNodes(content)[0];
  var insertedTo = xFoo.localDom.destinationInsertionPoints()[0];

  // the following should be true:
  assert.equal(distributed, div);
  assert.equal(insertedTo, content)

```

<a name="configure-method"></a>
## Configure callback

The `configure` method is part of an element's lifecycle and is automatically called 'top-down' and should be used to initialize default values for properties.  The function must return an object containing key/value pairs that will be used to set properties (keys) to default values.

Example:

```html
  configure: function() {
    // return default values of properties
    return {
        mode: 'auto',
        employees: []
    };
  }
```
In general, the configure method should only return the object containing default values, and not cause any side-effects on `this` that may interact with children, as these will still be in an un-configured state at this point.  Such actions should be done in the [ready callback](#ready-method).

<a name="ready-method"></a>
## Ready callback

The `ready` method is part of an element's lifecycle and is automatically called 'bottom-up' after the element's template has been stamped and all elements inside the element's local DOM have been `configure`'ed and had their `ready` method called. Implement `ready` when it's necessary to manipulate an element's local DOM when the element is constructed.

Example:

```html
  ready: function() {
    this.$.ajax.go();
  }
```

<a name="polymer-standard"></a>
# Polymer Standard Layer

<a name="node-marshalling"></a>
## Local node marshalling

Polymer automatically builds a map of instance nodes stamped into its local DOM, to provide convenient access to frequently used nodes without the need to query for (and memoize) them manually.  Any node specified in the element's template with an `id` is stored on the `this.$` hash by `id`.

Example:

```html
<dom-module id="x-custom">
    <template>
      Hello World from <span id="name"></span>!
    </template>
</dom-module>

<script>

  Polymer({

    is: 'x-custom',

    created: function() {
      this.$.name.textContent = this.name;
    }

  });

</script>
```

<a name="event-listeners"></a>
## Event listener setup

Event listeners can be added to the host element by providing an object-valued `listeners` property that maps events to event handler function names.

Example:

```html
<dom-module id="x-custom">
    <template>
      <div>I will respond</div>
      <div>to a click on</div>
      <div>any of my children!</div>
    </template>
</dom-module>

<script>

  Polymer({

    is: 'x-custom',

    listeners: {
      'click': 'handleClick'
    },
    
    handleClick: function(e) {
      alert("Thank you for clicking");
    }

  });

</script>
```

<a name="annotated-listeners"></a>
## Annotated event listener setup

For adding event listeners to local-DOM children, a more convenient `on-<event>` annotation syntax is supported directly in the template.  This often eliminates the need to give an element an `id` solely for the purpose of binding an event listener.

Example:

```html
<dom-module id="x-custom">
    <template>
      <button on-click="handleClick">Kick Me</button>
    </template>
</dom-module>

<script>

  Polymer({

    is: 'x-custom',

    handleClick: function() {
      alert('Ow!');
    }

  });

</script>
```

<a name="key-listeners"></a>
## Key listener setup

Polymer will automatically listen for `keydown` events and call handlers specified in the `keyPresses` object, which maps key codes to handler functions.  The key may either be specified as a keyboard code or one of several convenience strings supported:

* ESC_KEY
* ENTER_KEY
* LEFT
* UP
* RIGHT
* DOWN

Example:

```js
Polymer({

  is: 'x-custom',

  keyPresses: {
    'ESC_KEY': 'exitCurrentMode',
    88: 'handleXKeyPress'
  },

  exitCurrentMode: function(e) {
    ...
  },

  handleXKeyPress: function(e) {
    ...
  }

});
```

<a name="change-callbacks"></a>
## Property change callbacks

Custom element properties may be observed for changes by specifying an object-valued `bind` property that maps element properties to chagne handler names.  When the property changes, the change handler will be called with the new and old values.

Example:

```js
Polymer({

  is: 'x-custom',

  published: {
    disabled: Boolean
  },

  bind: {
    disabled: 'disabledChanged',
    highlight: 'highlightChanged'
  },

  disabledChanged: function(newValue, oldValue) {
    this.toggleClass('disabled', newValue);
    this.highlight = true;
  },

  highlightChanged: function() {
    this.classList.add('highlight');
    setTimeout(function() {
      this.classList.remove('highlight');
    }, 300);
  }

});
```

Note as in the example above, change handlers can be bound to properties that are not necessarily published.

Property change observation is achieved in Polymer by installing setters on the custom element prototype for properties with registered interest (as opposed to observation via Object.observe or dirty checking, for example).

Observing changes to object sub-properties is also supported via the `bind` object, by specifying a full (e.g. `user.manager.name`) or partial path (`user.*`).

Example:

```js
Polymer({

  is: 'x-custom',

  published: {
    user: Object
  },

  bind: {
    'user.manager.*': 'userManagerChanged'
  },

  userManagerChanged: function(newValue, oldValue, path) {
    if (path) {
      // sub-property of user.manager changed
      console.log('manager ' + path.split('.').pop() + ' changed to ' + newValue);
    } else {
      // user.manager object itself changed
      console.log('new manager name is ' + newValue.name);
    }
  }

});
```

Note that observing changes to paths (object sub-properties) is dependent on one of two requirements: either the value at the path in question changed via a Polymer [property binding](#property-binding) to another element, or the value was changed using the [`setPathValue`](#set-path) API, which provides the required notification to elements with registered interest.

<a name="property-binding"></a>
## Declarative property binding

### Basic property binding

Properties of the custom element may be bound into text content or properties of local DOM elements using binding annotations in the template.

To bind to textContent, the binding annotation must currently span the entire content of the tag:

```html
<dom-module id="user-view">
    <template>

      <!-- Supported -->
      First: <span>{{first}}</span><br>
      Last: <span>{{last}}</span>

      <!-- Not currently supported! -->
      <div>First: {{first}}</div>
      <div>Last: {{last}}</div>

    </template>
</dom-module>

<script>

  Polymer({

    is: 'user-view',

    published: {
      first: String,
      last: String
    }

  });

</script>

<user-view firstName="Samuel" lastName="Adams"></user-view>

```

To bind to properties, the binding annotation should be provided as the value to an attribute with the same name of the JS property to bind to:

```html
<dom-module id="main-view">
    <template>
      <user-view firstName="{{user.first}}" last="{{user.last}}"></user-view>
    </template>
</dom-module>

<script>

  Polymer({

    is: 'main-view',

    published: {
      user: Object
    }

  });

</script>
```

As in the exmaple above, paths to object sub-properties may also be specified in templates.  See [Binding to structured data](#path-binding) for details.

Note that while HTML attributes are used to specify bindings, values are assigned directly to JS properties, not to the HTML attributes of the elements.

Note that currently binding to `style` is a special case which results in the value being set to `style.cssText`.

<a name="property-notification"></a>
### Property change notification and Two-way binding

Polymer supports cooperative two-way binding between elements, allowing elements that "produce" data or changes to data to propagate those changes upwards to hosts when desired.

When a Polymer elements changs a property that was "published" as part of its public API with the `notify` flag set to true, it automatically fires a non-bubbling DOM event to indicate those changes to interested hosts.  These events follow a naming convention of `<property>-changed`, and contain a `value` property in the `event.detail` object indicating the new value.

As such, one could attach an `on-<property>-changed` listener to an element to be notified of changes to such properties, set the `event.detail.value` to a property on itself, and take necessary actions based on the new value.  However, given this is a common pattern, bindings using "curly-braces" (e.g. `{{property}}`) will automatically perform this upwards binding automatically without the user needing to perform those tasks.  This can be defeated by using "square-brace" syntax (e.g. `[[property]]`), which results in only one-way (downward) data-binding.

To summarize, two-way data-binding is achieved when both the host and the child agree to participate, satisfying these three conditions:

1. The host must use curly-brace `{{property}}` syntax.  Square-brace `[[property]]` syntax results in one-way downward binding, regardless of the notify state of the child's property.
2. The child property being bound to must be published with the `notify` flag set to true (or otherwise send a `<propety>-changed` custom event).  If the property being bound is not published or if the `notify` flag is not set, only one-way (downward) binding will occur.
3. The child property being bound to must not published with the `readOnly` flag set to true.  If the child property is `notify: true` and `readOnly:true`, and the host binding uses curly-brace syntax, the binding will effectively be one-way (upward).

Example 1: Two-way binding

```html

<script>
  Polymer({
    is: 'custom-element',
    published: {
      prop: {
        type: String,
        notify: true
      }
    }
  });
</script>

...

<!-- changes to `value` propagate downward to `prop` on child -->
<!-- changes to `prop` propagate upward to `value` on host  -->
<custom-element prop="{{value}}"></custom-element>
```

Example 2: One-way binding (downward)

```html

<script>
  Polymer({
    is: 'custom-element',
    published: {
      prop: {
        type: String,
        notify: true
      }
    }
  });
</script>

...

<!-- changes to `value` propagate downward to `prop` on child -->
<!-- changes to `prop` are ignored by host due to square-bracket syntax -->
<custom-element prop="[[value]]"></custom-element>
```

Example 3: One-way binding (downward)

```html

<script>
  Polymer({
    is: 'custom-element',
    published: {
      prop: String    // no `notify:true`!
    }
  });
</script>

...

<!-- changes to `value` propagate downward to `prop` on child -->
<!-- changes to `prop` are not notified to host due to notify:falsey -->
<custom-element prop="{{value}}"></custom-element>
```

Example 4: One-way binding (upward)

```html

<script>
  Polymer({
    is: 'custom-element',
    published: {
      prop: {
          type: String,
          notify: true,
          readOnly: true
        }
    }
  });
</script>

...

<!-- changes to `value` are ignored by child due to readOnly:true -->
<!-- changes to `prop` propagate upward to `value` on host  -->
<custom-element prop="{{value}}"></custom-element>
```

Example 5: Error / non-sensical state

```html

<script>
  Polymer({
    is: 'custom-element',
    published: {
      prop: {
          type: String,
          notify: true,
          readOnly: true
        }
    }
  });
</script>

...

<!-- changes to `value` are ignored by child due to readOnly:true -->
<!-- changes to `prop` are ignored by host due to square-bracket syntax -->
<!-- binding serves no purpose -->
<custom-element prop="[[value]]"></custom-element>
```

<a name="path-binding"></a>
### Binding to structured data

Sub-properties of objects may be two-way bound to properties of custom elements as well by specifying the path of interest to the binding annotation.

Example:

```html
<template>
  <div>{{user.manager.name}}</div>
  <user-element user="{{user}}"></user-element>
</template>
```

As with change handlers for paths, bindings to paths (object sub-properties) are dependent on one of two requirements: either the value at the path in question changed via a Polymer [property binding](#property-binding) to another element, or the value was changed using the [`setPathValue`](#set-path) API, which provides the required notification to elements with registered interest, as discussed below.

Note that path bindings are distinct from property bindings in a subtle way: when a property's value changes, an assignment must occur for the value to propagate to the property on the element at the other side of the binding.  However, if two elements are bound to the same path of a shared object and the value at that path changes (via a property binding or via `setPathValue`), the value seen by both elements actually changes with no additional assignment necessary, by virtue of it being a property on a shared object reference.  In this case, the element who changed the path must notify the system so that other elements who have registered interest in the same path may take side effects.  However, there is no concept of one-way binding in this case, since there is no concept of propagation.  That is, all bindings and change handlers for the same path will always be notified and update when the value of the path changes.

<a name="set-path"></a>
### Path change notification

Two-way data-binding and observation of paths in Polymer is achieved using a similar strategy to the one described above for [2-way property binding](#property-notification): When a sub-property of a published `Object` changes, an element fires a non-bubbling `<property>-path-changed` DOM event with a `detail.path` value indicating the path on the object that changed.  Elements that have registered interest in that object (either via binding or change handler) may then take side effects based on knowledge of the path having changed.  Finally, those elements will forward the notification on to any children they have bound the object to, and if the element published the root object for the path that changed on its API, it will also fire a new `<propety>-path-changed` event appropriately.  Through this method, a notification will reach any part of the tree that has registered interest in that path so that side effects occur.

This system "just works" to the extent that changes to object sub-properties occur as a result of being bound to a notifying custom element property that changed.  However, often imperative code needs to "poke" at an object's sub-properties directly.  As we avoid more sophisticated observation mechanisms such as Object.observe or dirty-checking in order to achieve the best startup and runtime performance cross-platform for the most common use cases, changing an object's sub-properties directly requires cooperation from the user.

Specifically, Polymer provides two API's that allow such changes to be notified to the system: `notifyPath(path, value)` and `setPathValue(path, value)`.

Example:

```html
<dom-module id="custom-element">
    <template>
      <div>{{user.manager.name}}</div>
    </template>
</dom-module>

<script>
  Polymer({

    is: 'custom-element',

    reassignManager: function(newManager) {
      this.user.manager = newManager;
      // Notification required for binding to update!
      this.notifyPath('user.manager', this.user.manager);
    }

  });
</script>
```

Since in the majority of cases, notifyPath will be called directly after an assignment, a convenience function `setPathValue` is provided that performs both actions:

```js
    reassignManager: function(newManager) {
      this.setPathValue('user.manager', newManager);
    }
```

### Expressions in binding annotations

Currently the only binding expression supported in Polymer binding annotations is negation using `!`:

Example:

```html
<template>
  <div hidden="{{!enabled}}"></div>
</template>
```

<a name="attribute-binding"></a>
## Declarative attribute binding

In the vast majority of cases, binding data to other elements should use property binding described above, where changes are propagated by setting the new value to the JavaScript property on the element.

However, there may be cases where a user actually needs to set an attribute on an element, as opposed to a property.  These include when attribute selectors are used for CSS or for for interoperability with elements that require using attribute-based API.

Polymer provides an alternate binding annotation syntax to make it explicit when binding values to attributes is desired by using `$=` rather than `=`.  This results in in a call to `element.setAttribute('<attr>', value);`, as opposed to `element.property = value;`.

```html
<template>

    <!-- Attribute binding -->
    <my-element selected$="{{value}}"></my-element>    
    <!-- results in <my-element>.setAttribute('selected', this.value); -->

    <!-- Property binding -->
    <my-element selected="{{value}}></my-element>
    <!-- results in <my-element>.selected = this.value; -->

</template>
```
Values will be serialized according to type: Arrays/Objects will be `JSON.stringify`'ed, booleans will result in a non-valued attribute to be either set or removed, and `Dates` and all primitive types will be serialized using the value returned from `toString`.

Again, as values must be serialized to strings when binding to attributes, it is always more performant to use property binding for pure data propagation.

<a name="attribute-reflection"></a>
## Reflecting properties to attributes

In specific cases, it may be useful to keep an HTML attribute value in sync with a property value.  This may be achieved by setting `reflect: true` on a property in the published configuration object.  This will cause any change to the property to be serialized out to an attribute of the same name.

```html
<script>
    Polymer({
    
       published: {
         response: {
            type: Object,
            reflect: true
         }
       },
    
       responseHandler: function(response) {
         this.response = 'loaded';
         // results in this.setAttribute('response', 'loaded');
       }
    
    });
</script>
```

Values will be serialized according to type: Arrays/Objects will be `JSON.stringify`'ed, booleans will result in a non-valued attribute to be either set or removed, and `Dates` and all primitive types will be serialized using the value returned from `toString`.

<a name="computed-properties"></a>
## Computed properties

Polymer supports virtual properties whose values are calculated from other properties.  Computed properties can be defined by providing an object-valued `computed` property on the prototype that maps property names to computing functions.  The name of the function to compute the value is provided as a string with dependent properties as arguments in parenthesis.  Only one dependency is supported at this time.

```html
<dom-module id="x-custom">
    <template>
      My name is <span>{{fullName}}</span>
    </template>
<dom-module id="x-custom">

<script>
    Polymer({
    
       is: 'x-custom',
    
       computed: {
         // when `user` changes `computeFullName` is called and the
         // value it returns is stored as `fullName`
         fullName: 'computeFullName(user)',
       },

       computeFullName: function(user) {
         return user.firstName + ' ' + user.lastName;
       }

      ...

    });
</script>
```

<a name="read-only"></a>
## Read-only properties

When a property only "produces" data and never consumes data, this can be made explicit to avoid accidental changes from the host by setting the `readOnly` flag to `true` in the published property definition.  In order for the element to actually change the value of the property, it must use a private generated setter of the convention `_set<Property>(value)`.

```html
<script>
    Polymer({
    
       published: {
         response: {
            type: Object,
            readOnly: true,
            notify: true
         }
       },
    
       responseHandler: function(response) {
         this._setResponse(response);
       }
    
      ...
    
    });
</script>
```

Generally, read-only properties should also be set to `notify: true` such that their changes are observable from above.

<a name="utility-functions"></a>
## Utility Functions

Polymer's Base prototype provides a set of useful convenience/utility functions for instances to use.  See API documentation for more details.

* toggleClass: function(name, bool, [node])
* toggleAttribute: function(name, bool, [node])
* attributeFollows: function(name, neo, old)
* fire: function(type, [detail], [onNode], [bubbles], [cancelable])
* async: function(method)
* queryHost: function(node)
* transform: function(node, transform)
* translate3d: function(node, x, y, z)
* importHref: function(href, onload, onerror)

<a name="layout-html"></a>
## Attribute-based layout

CSS is notoriously verbose for doing even the most basic layout tasks, and so Polymer provides a useful set of layout CSS that can be applied using very terse HTML attribues (as opposed to CSS classes), which we find greatly improves the readability of markup.  Below is a quick cheatsheet of attributes available; refer to `layout.html` directly for details.

General:

* block
* hidden
* relative
* fit
* fullbleed

Flexbox:

* layout horizontal, layout vertical
  * inline
  * reverse
  * wrap
  * wrap-reverse
  * start
  * center
  * end
  * start-justified
  * center-justified
  * end-justified
  * around-justified
  * center-center
  * justified

Flexbox children:

* flex
  * auto
  * none
  * one .. twelve
* self-start
* self-center
* self-end
* self-stretch

<a name="migration-notes"></a>
# Migration Notes

This section covers how to deal with yet-unimplemented and/or de-scoped features in Polymer 0.8 as compared to 0.5.  Many of these are simply un-implemented; that is, we will likely have a final "solution" that addresses the need, we just haven't tackled that feature yet as we address items in priority order.  Other solutions in 0.8 may be lower-level as compared to 0.5, and will be explained here.

As the final 0.8 API solidifies, this section will be updated accordingly.  As such, this section should be considered answers "how do I solve problem xyz <em>TODAY</em>", rather than a representation of the final Polymer 0.8 API.

## Property casing

**Warning:** Currently only lower-case published properties are supported.  Support for upper-case properties will be added this sprint.  In the short-term, please use lower-case properties.

## Styling

TODO: explain shadow/shady DOM styling considerations.

* `<style>` goes outside the template
* Prefix with element name

```css
x-foo .my-class {
  ...
}
```

## Self / Child Configuration

Lifecycle callback timing and best practices are in high flux at the moment.

__TL;DR: Do all initialization/configuration in `ready` today.__

Currently, at `created` time, children are not stamped yet.  As such, configuring any properties that may have side-effects involving children will error.  As such, it is not reccomended to use the `created` callback for self-configuration.

There is a work-in-progress `configure` callback that is called top-down after children have been stamped & `created` , but is not ready for use yet.

The `ready` callback is called bottom-up after children have been `configure`-ed.  Currently, this is the only useful & safe place to do configuration of properties that may have side-effects on children.  Ideally this configuration step will be moved to the `configure` callback in the future.


## Binding limitations

Current limitations that are on the backlog for evaluation/improvement are listed below, with current workarounds:

* Sub-textContent binding
    * Use `<span>`'s to break up textContent into discrete elements
* CSS class binding:
    * May bind entire class list from one property to `class` _attribute_:
      `<div class$="{{classes}}">`
    * Otherwise, `this.classList.add/remove` from change handlers
* CSS inline-style binding:
    * May bind entire inline style from one property to `style` _property_:
      `<div style="{{styles}}">`
    * Otherwise, assign `this.style.props` from change handlers
* Support for compound property binding
    * See below

## Compound property effects

Polymer 0.8 currently has no built-in support for compound observation or compound binding expressions.  This problem space is on the backlog to be tackled in the near future.  This section will discuss lower-level tools that are available in 0.8 that can be used instead.

Assume an element has a boolean property that should be set when either of two conditions are true: e.g. when `<my-parent>.isManager == true` OR `<my-parent>.mode == 2`, you want to set `<my-child>.disabled = true`.

The most naive way to achieve this in 0.8 is with separate change handlers for the dependent properties that set a `shouldDisable` property bound to the `my-child`.

Example:

```html
<dom-module id="x-parent">
    <template>
        <x-child disabled="{{shouldDisable}}"></my-child>
    </template>
</dom-module>

<script>
Polymer({
    
    is: 'x-parent',
    
    bind: {
        isManager: 'computeShouldDisable',
        mode: 'computeShouldDisable',
    },

    // Warning: Called once for every change to dependent properties!
    computeShouldDisable: function() {
        this.shouldDisable = this.isManager || (this.mode == 2);
    }

});
</script>
```

Due to the synchronous nature of bindings in 0.8, code such as the following will result in `<my-child>.disabled` being set twice (and any side-effects of that property changing to potentially occur twice):

```js
myParent.isManager = false;
myParent.mode = 5;
```

If the work of computing the property is expensive, or if the side-effects of the binding are expensive, then you may want to ensure side-effects only occur once for any number of changes to them during a turn by manually introducing asynchronicity.  The `debounce` API on the Polymer Base prototype can be used to achieve this.  The `debounce` API takes a signal name (String), callback, and optional wait time, and only calls the callback once for any number `debounce` calls with the same `signalName` started within the wait period.

Example:

```html
<dom-module id="x-parent">
    <template>
        <x-child disabled="{{shouldDisable}}"></my-child>
    </template>
</dom-module>

<script>
Polymer({
    
    is: 'x-parent',
    
    bind: {
        isManager: 'computeShouldDisableDebounced',
        mode: 'computeShouldDisableDebounced',
    },

    computeShouldDisableDebounced: function() {
        this.debounce('computeShouldDisable', this.computeShouldDisable);
    },

    // Better: called once for multiple changes
    computeShouldDisable: function() {
        this.shouldDisable = this.isManager || (this.mode == 2);
    }

});
</script>
```
Thus, for the short term we expect users will need to consider compound effects and apply use of the `debounce` function to ensure efficient side-effects.

## Structured data and path notification

TODO - call `setPathValue` and/or `notifyPath` to notify non-bound path changes

## Repeating elements

Repeating templates is moved to a custom element (HTMLTemplateElement type extension called `x-repeat`):

```html
<template is="x-repeat" items="{{data}}">
    <div>{{item.sub}}</div>
</template>
```

## Array notification

TODO - array changes not observed; for now need to "kick" x-repeat's `render`

<a name="todo-inheritance"></a>
## Mixins / Inheritance

TODO - use composition for now

## Gesture support

TODO - use standard DOM for now until gesture support is ported