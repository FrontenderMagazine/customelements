# Custom Elements: Defining new elements in HTML

**Heads up!** This article discusses APIs that are not yet fully standardized and still in flux. Be cautious when using experimental APIs in your own projects.

## Introduction

The web severely lacks expression. To see what I mean, take a peek at a "modern" web app like GMail:

![Gmail][1] Modern web apps: built with `<div>` soup.

There's nothing modern about `<div>` soup. And yet,…this is how we build web apps. It's sad. Shouldn't we demand more from our platform?

### Sexy markup. Let's make it a thing.

HTML gives us an excellent tool for structuring a document but its vocabulary is limited to elements the [HTML standard][2] defines.

What if the markup for GMail wasn't atrocious? What if it was beautiful:

    <hangout-module>
      <hangout-chat from="Paul, Addy">
        <hangout-discussion>
          <hangout-message from="Paul" profile="profile.png"
              profile="118075919496626375791" datetime="2013-07-17T12:02">
            <p>Feelin' this Web Components thing.</p>
            <p>Heard of it?</p>
          </hangout-message>
        </hangout-discussion>
      </hangout-chat>
      <hangout-chat>...</hangout-chat>
    </hangout-module>

[Try the demo!][3]

How refreshing! This app totally makes sense too. It's **meaningful**, **easy to understand**, and best of all, it's **maintainable**. Future me/you will know exactly what it does just by examining its declarative backbone.

Help us custom elements. You're our only hope!

## Getting started

[Custom Elements][4] **allow web developers to define new types of HTML elements**. The spec is one of several new API primitives landing under the [Web Components][5] umbrella, but it's quite possibly the most important. Web Components don't exist without the features unlocked by custom elements:

  1. Define new HTML/DOM elements
  2. Create elements that extend from other elements
  3. Logically bundle together custom functionality into a single tag
  4. Extend the API of existing DOM elements

### Registering new elements

Custom elements are created using `document.register()`:

    var XFoo = document.register('x-foo');
    document.body.appendChild(new XFoo());

The first argument to `document.register()` is the element's tag name. The name **must contain a dash (-)**. So for example, `x-tags`, `my-element`, and `my-awesome-app` are all valid names, while `tabs` and `foo_bar` are not. This restriction allows the parser to distinguish custom elements from regular elements but also ensures forward compatibility when new tags are added to HTML.

The second argument is an (optional) object describing the element's `prototype`. This is the place to add custom functionality (e.g. public properties and methods) to your elements. [More on that][6] later.

By default, custom elements inherit from `HTMLElement`. Thus, the previous example is equivalent to:

    var XFoo = document.register('x-foo', {
      prototype: Object.create(HTMLElement.prototype)
    });

A call to `document.register('x-foo')` teaches the browser about the new element, and returns a constructor that you can use to create instances of `x-foo`. Alternatively, you can use the other [techniques of instantiating elements][7] if you don't want to use the constructor.

If it's undesirable that the constructor ends up on the global `window` object, put it in a namespace (`var myapp = {}; myapp.XFoo = document.register('x-foo');`) or drop it on the floor.

### Extending native elements

Say you aren't happy with Regular Joe™ `button`. You'd like to supercharge its capabilities to be a "Mega Button". To extend the `button` element, create a new element that inherits the `prototype` of `HTMLButtonElement`:


    var MegaButton = document.register('mega-button', {
      prototype: Object.create(HTMLButtonElement.prototype)
    });


To create **element A** that extends **element B**, **element A** must inherit the `prototype` of **element B**.

Custom elements like this are called _type extension custom elements_. They inherit from a specialized version of `HTMLElement` as a way to say, "element X is a Y".

Example:

    <button is="mega-button">


### How elements are upgraded

Have you ever wondered why the HTML parser doesn't throw a fit on non-standard tags? For example, it's perfectly happy if we declare `<randomtag>` on the page. According to the [HTML specification][8]:

The `HTMLUnknownElement` interface must be used for HTML elements that are not defined by this specification.

Sorry `randomtag`! You're non-standard and inherit from `HTMLUnknownElement`.

The same is not true for custom elements. **Elements with valid custom element names inherit from `HTMLElement`.** You can verify this fact by firing up the Console: Ctrl+Shift+J (or Cmd+Opt+J on Mac), and paste in the following lines of code; they return `true`:


    // "tabs" is not a valid custom element name
    document.createElement('tabs').__proto__ === HTMLUnknownElement.prototype

    // "x-tabs" is a valid custom element name
    document.createElement('x-tabs').__proto__ == HTMLElement.prototype


**Note:** `<x-tabs>` will still be an `HTMLUnknownElement` in browsers that don't support `document.register()`.

#### Unresolved elements

Because custom elements are registered by script using `document.register()`, **they can be declared or created _before_ their definition is registered** by the browser. For example, you can declare `x-tabs` on the page but end up invoking `document.register('x-tabs')` much later.

Before elements are upgraded to their definition, they're called **unresolved elements**. These are HTML elements that have a valid custom element name but haven't been registered.

This table might help keep things straight:

<table>
<tr><td>Name</td><td>Inherits from</td><td>Examples</td></tr>
<tr><td>Unresolved element</td><td>HTMLElement</td><td>`<x-tabs>`, `<my-element>`, `<my-awesome-app>`</td></tr>
<tr><td>Unknown element</td><td>HTMLUnknownElement</td><td>`<tabs>`, `<foo_bar>`</td></tr>
</table>

## Instantiating elements

The common techniques of creating elements still apply to custom elements. As with any standard element, they can be declared in HTML or created in DOM using JavaScript.

### Instantiating custom tags

**Declare** them:

    <x-foo></x-foo>


**Create DOM** in JS:


    var xFoo = document.createElement('x-foo');
    xFoo.addEventListener('click', function(e) {
      alert('Thanks!');
    });


Use the **`new` operator**:


    var xFoo = new XFoo();
    document.body.appendChild(xFoo);


### Instantiating type extension elements

Instantiating type extension-style custom elements is strikingly close to custom tags.

**Declare** them:

    <!-- <button> "is a" mega button -->
    <button is="mega-button">


**Create DOM** in JS:


    var megaButton = document.createElement('button', 'mega-button');
    // megaButton instanceof MegaButton === true


As you can see, there's now an overloaded version of `document.createElement()` that takes the `is=""` attribute as its second parameter.

Use the **`new` operator**:


    var megaButton = new MegaButton();
    document.body.appendChild(megaButton);


So far, we've learned how to use `document.register()` to tell the browser about a new tag…but it doesn't do much. Let's add properties and methods.

## Adding JS properties and methods

The powerful thing about custom elements is that you can bundle tailored functionality with the element by defining properties and methods on the element definition. Think of this as a way to create a public API for your element.

Here's a full example:


    var XFooProto = Object.create(HTMLElement.prototype);

    // 1. Give x-foo a foo() method.
    XFooProto.foo = function() {
      alert('foo() called');
    };

    // 2. Define a property read-only "bar".
    Object.defineProperty(XFooProto, "bar", {value: 5});

    // 3. Register x-foo's definition.
    var XFoo = document.register('x-foo', {prototype: XFooProto});

    // 4. Instantiate an x-foo.
    var xfoo = document.createElement('x-foo');

    // 5. Add it to the page.
    document.body.appendChild(xfoo);


Of course there are umpteen thousand ways to construct a `prototype`. If you're not a fan of creating prototypes like this, here's a more condensed version of the same thing:


    var XFoo = document.register('x-foo', {
      prototype: Object.create(HTMLElement.prototype, {
        bar: {
          get: function() { return 5; }
        },
        foo: {
          value: function() {
            alert('foo() called');
          }
        }
      })
    });


The first format allows for the use of ES5 `[Object.defineProperty`][9]. The second allows the use of [get/set][10].

### Lifecycle callback methods

Elements can define special methods for tapping into interesting times of their existence. These methods are appropriately named the **lifecycle callbacks**. Each has a specific name and purpose:

<table>
<tr><td>Callback name</td><td>Called when</td></tr>

<tr><td>createdCallback</td>
<td>an instance of the element is created</td></tr>

<tr><td>enteredDocumentCallback</td>
<td>an instance was inserted into the document</td></tr>

<tr><td>leftDocumentCallback</td>
<td>an instance was removed from the document</td></tr>

<tr><td>attributeChangedCallback(attrName, oldVal, newVal)</td>
<td>an attribute was added, removed, or updated</td></tr>
</table>

**Example:** defining `createdCallback()` and `enteredDocumentCallback()` on `x-foo`:

    var proto = Object.create(HTMLElement.prototype);

    proto.createdCallback = function() {...};
    proto.enteredDocumentCallback = function() {...};

    var XFoo = document.register('x-foo', {prototype: proto});


**All of the lifecycle callbacks are optional**, but define them if/when it makes sense. For example, say your element is sufficiently complex and opens a connection to IndexedDB in `createdCallback()`. Before it gets removed from the DOM, do the necessary cleanup work in `leftDocumentCallback()`. **Note:** you shouldn't rely on this, for example, if the user closes the tab, but think of it as a possible optimization hook.

Another use case lifecycle callbacks is for setting up default event listeners on the element:


    proto.createdCallback = function() {
      this.addEventListener('click', function(e) {
        alert('Thanks!');
      });
    };


## Adding markup

We've created `x-foo`, given it a JavaScript API, but it's blank! Shall we give it some HTML to render?

[Lifecycle callbacks][11] come in handy here. Particularly, we can use `createdCallback()` to endow an element with some default HTML:


    var XFooProto = Object.create(HTMLElement.prototype);

    XFooProto.createdCallback = function() {
      this.innerHTML = "I'm an x-foo-with-markup!";
    };

    var XFoo = document.register('x-foo-with-markup', {prototype: XFooProto});


Instantiating this tag and inspecting in the DevTools (right-click, select Inspect Element) should show:

    ▾<x-foo-with-markup>
       <b>I'm an x-foo-with-markup!</b>
     </x-foo-with-markup>


### Encapsulating the internals in Shadow DOM

By itself, [Shadow DOM][12] is a powerful tool for encapsulating content. Use it in conjunction with custom elements and things get magical!

Shadow DOM gives custom elements:

  1. A way to hide their guts, thus shielding users from gory implementation details.
  2. [Style encapsulation][13]…fo' free.

Creating an element from Shadow DOM is like creating one that renders basic markup. The difference is in `createdCallback()`:


    var XFooProto = Object.create(HTMLElement.prototype);

    XFooProto.createdCallback = function() {
      // 1. Attach a shadow root on the element.
      var shadow = this.createShadowRoot();

      // 2. Fill it with markup goodness.
      shadow.innerHTML = "I'm in the element's Shadow DOM!";
    };

    var XFoo = document.register('x-foo-shadowdom', {prototype: XFooProto});


Instead of setting the element's `.innerHTML`, I've created a Shadow Root for `x-foo-shadowdom` and then filled it with markup. With the "Show Shadow DOM" setting enabled in the DevTools, you'll see a `#document-fragment` that can be expanded:

    ▾<x-foo-shadowdom>
       ▾#document-fragment
         <b>I'm in the element's Shadow DOM!</b>
     </x-foo-shadowdom>

That's the Shadow Root!

### Creating elements from a template

[HTML Templates][14] are another new API primitive that fits nicely into the world of custom elements.

For those unfamiliar, the `<template>` element][15] allows you to declare fragments of DOM which are parsed, inert at page load, and instantiated later at runtime. They're an ideal placeholder for declaring the structure of custom element.

**Example:** registering an element created from a `<template>` and Shadow DOM:

    <template id="sdtemplate">
      <style>
        p { color: orange; }
      </style>
      <p>I'm in Shadow DOM. My markup was stamped from a &lt;template&gt;.</p>
    </template>

    <script>
    var proto = Object.create(HTMLElement.prototype, {
      createdCallback: {
        value: function() {
          var t = document.querySelector('#sdtemplate');
          this.createShadowRoot().appendChild(t.content.cloneNode(true));
        }
      }
    });
    document.register('x-foo-from-template', {prototype: proto});
    </script>


I'm in Shadow DOM. My markup was stamped from a `<template>`.

These few lines of code pack a lot of punch. Let's understanding everything that is happening:

  1. We've registered a new element in HTML: `<template>`
  2. The element's DOM was created from a `<template>`
  3. The element's scary details are hidden away using Shadow DOM
  4. Shadow DOM gives the element style encapsulation (e.g. `p {color: orange;}` isn't turning the entire page orange)

So good!

## Styling custom elements

As with any HTML tag, users of your custom tag can style it with selectors:

    <style>
      app-panel {
        display: flex;
      }
      [is="x-item"] {
        transition: opacity 400ms ease-in-out;
        opacity: 0.3;
        flex: 1;
        text-align: center;
        border-radius: 50%;
      }
      [is="x-item"]:hover {
        opacity: 1.0;
        background: rgb(255, 0, 255);
        color: white;
      }
      app-panel > [is="x-item"] {
        padding: 5px;
        list-style: none;
        margin: 0 7px;
      }
    </style>

    <app-panel>
      <li is="x-item">Do</li>
      <li is="x-item">Re</li>
      <li is="x-item">Mi</li>
    </app-panel>

### Styling elements that use Shadow DOM

The rabbit hole goes much _much_ deeper when you bring Shadow DOM into the mix. [Custom elements that use Shadow DOM][16] inherit its great benefits.

Shadow DOM infuses an element with style encapsulation. Styles defined in a Shadow Root don't leak out of the host and don't bleed in from the page. **In the case of a custom element, the element itself is the host.** The properties of style encapsulation also allow custom elements to define default styles for themselves.

Shadow DOM styling is a huge topic! If you want to learn more about it, I recommend a few of my other articles:

### FOUC prevention using :unresolved

To mitigate [FOUC][17], custom elements spec out a new CSS pseudo class, `:unresolved`. Use it to target [unresolved elements][18], right up until the point where the browser invokes your `createdCallback()` (see [lifecycle methods][11]). Once that happens, the element is no longer an unresolved element. The upgrade process is complete and the element has transformed into its definition.

CSS `:unresolved` is supported natively in Chrome 29.

**Example**: fade in "x-foo" tags when they're registered:

    <style>
      x-foo {
        opacity: 1;
        transition: opacity 300ms;
      }
      x-foo:unresolved {
        opacity: 0;
      }
    </style>

Keep in mind that `:unresolved` only applies to [unresolved elements][18], not to elements that inherit from `HTMLUnkownElement` (see [How elements are upgraded][19]).

    <style>
      /* apply a dashed border to all unresolved elements */
      :unresolved {
        border: 1px dashed red;
        display: inline-block;
      }
      /* x-panel's that are unresolved are red */
      x-panel:unresolved {
        color: red;
      }
      /* once the definition of x-panel is registered, it becomes green */
      x-panel {
        color: green;
        display: block;
        padding: 5px;
        display: block;
      }
    </style>

    <panel>
      I'm black because :unresolved doesn't apply to "panel".
      It's not a valid custom element name.
    </panel>

    <x-panel>I'm red because I match x-panel:unresolved.</x-panel>

I'm black because :unresolved doesn't apply to "panel". It isn't a valid name.I'm red because I match x-panel:unresolved.

Register

For more on `:unresolved`, see Polymer's [A Guide to styling elements][20].

## History and browser support

### Feature detection

Feature detecting is a matter of checking if `document.register()` exists:


    function supportsCustomElements() {
      return 'register' in document;
    }

    if (supportsCustomElements()) {
      // Good to go!
    } else {
      // Use other libraries to create components.
    }


### Browser support

`document.register()` first started landing behind a flag in Chrome 27 and Firefox ~23. However, the specification has evolved quite a bit since then. Chrome 31 is the first to have true support for the updated spec.

Custom elements can be enabled in Chrome 31 under "Experimental Web Platform features" in `about:flags`.

Until browser support is stellar, there are a couple of great polyfills:

### What happened to HTMLElementElement?

For those that have followed the standardization work, you know there was once `<element>`. It was the bees knees. You could use it to declaratively register new elements:

    <element name="my-element">
      ...
    </element>

Unfortunately, there were too many timing issues with the [upgrade process][19], corner cases, and Armageddon-like scenarios to work it all out. `element` had to be shelved. In August 2013, Dimitri Glazkov posted to [public-webapps][21] announcing its removal, at least for now.

It's worth noting that Polymer implements a declarative form of element registration with `<polymer-element>`. How? It uses `document.register('polymer-element')` and the techniques I described in [Creating elements from a template][22].

## Conclusion

Custom elements give us the tool to extend HTML's vocabulary, teach it new tricks, and jump through the wormholes of the web platform. Combine them with the other new platform primitives like Shadow DOM and `<template>`, and we start to realize the picture of Web Components. Markup can be sexy again!

If you're interested in getting started with web components, I recommend checking out [Polymer][23]. It's got more than enough to get you going.

   [1]: img/gmail.png
   [2]: http://www.whatwg.org/specs/web-apps/current-work/multipage/
   [3]: https://html5-demos.appspot.com/static/webcomponents-bdconf/demos/components/hangouts/index.html
   [4]: https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/custom/index.html
   [5]: https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html
   [6]: http://www.html5rocks.com#publicapi
   [7]: http://www.html5rocks.com#instantiating
   [8]: http://www.whatwg.org/specs/web-apps/current-work/multipage/elements.html#htmlunknownelement
   [9]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
   [10]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/get
   [11]: http://www.html5rocks.com#lifecycle
   [12]: http://www.html5rocks.com/tutorials/webcomponents/shadowdom/
   [13]: http://www.html5rocks.com/tutorials/webcomponents/shadowdom-201/
   [14]: https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/templates/index.html
   [15]: http://www.html5rocks.com/tutorials/webcomponents/template/
   [16]: http://www.html5rocks.com#shadowdom
   [17]: http://en.wikipedia.org/wiki/Flash_of_unstyled_content
   [18]: http://www.html5rocks.com#unresolvedels
   [19]: http://www.html5rocks.com#upgrades
   [20]: http://www.polymer-project.org/articles/styling-elements.html#preventing-fouc
   [21]: http://lists.w3.org/Archives/Public/public-webapps/2013JulSep/0287.html
   [22]: http://www.html5rocks.com#fromtemplate
   [23]: http://www.html5rocks.com/polymer-project.org
