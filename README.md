# Polymer Module Loader

PML is an implementation of the
[AMD specification](https://github.com/amdjs/amdjs-api/blob/master/AMD.md).
The primary goal of it is to play nice with HTML Imports. Based on [IMD](https://github.com/PolymerLabs/IMD).

## Why PML?

JavaScript module systems generally perform three different tasks:

  1. **Definition** Allow module authors to define an encapsulated unit of code,
     including what other modules it depends on.
  2. **Resolving** Given a module reference, resolve it to an actual module
     instance. This typically involves a central module registry that maps
     module names to instances.
  3. **Loading** Given a module reference that is not yet loaded into the
     registry, load it from some source, usually a URL (via an XHR).

Together, PML and HTML Imports form a really great module system.

## What about ES6 (ES2015) Modules?

They're great. Really, we can't wait for them to be implemented natively in
browsers. And until then, compilers and loaders like SystemJS and Babel do
allow developers to use ES6 modules now, _if_ they're willing to deal with a
build step.

Not every project can or wishes to do so, and HTML Imports is not a JavaScript
module definition system, so PML is an extremely lightweight way to use modules
on top of imports.

When the ES6 module loader spec is further along, we will look into ways of
integrating the two so that JavaScript can import HTML "modules" and vice-versa.
We're very excited about the possibilities for interop.

## How Do I?

### Download and Include PML

PML is distributed as an HTML Import-able HTML file, naturally. Install and uselike this:

Install:
```
> bower install panuhorsmalahti/PML --save
```

Import:

```html
<link rel="import" href="../pml/pml.html">
```

We recommend that HTML files that require or define _do_ _not_ directly import
PML, but rather let the main page import PML, usually as the very first import.
This is so that an application that really requires RequireJS to load modules
(this can be true when loading pure JS modules with no corresponding `<script>`
tags), can setup RequireJS instead of PML. Since PML is fully AMD compliant,
all modules defined in HTML imports will work just fine.

### Define a Module

Modules are defined exactly as in other AMD systems like RequireJS:
Public modules are defined by name:

Here's the definition of a mythical module, 'squidbits', that depends on the
modules 'tentacles.html', and 'ink':

```javascript
define('squidbits', ['./tentacles.html', 'ink'], function(tentacles, ink) {
  return {tentacles: tentacles, ink: ink, squidbits: true};
});
```

Now, since PML is targeted at projects already using HTML Imports, it's likely
that the module will be defined inside an HTML file, like so:

```html
<link rel="import" href="./tentacles.html">
<script src="ink.js"></script>

<script>
define('squidbits', ['./tentacles.html', 'ink'], function(tentacles, ink) {
  return {tentacles: tentacles, ink: ink, squidbits: true};
});
</script>
```

Notice the `<link>` and `<script>` tags. These tell the browser to load
'tentacles.html' and 'ink.js'. Because scripts are blocked until imports and
other scripts have loaded and run, and 'tentacles.html' and 'ink.js' define
modules, `squidbits`'s dependencies are guaranteed to be loaded and registered
when the inline script runs.

_Note that we're assuming that `ink.js` defines a `ink` module here, and
`tentacles.html` defines an anonymous module._

## Define Anonymous / Relative Modules

Private (relative) module names are inferred from the current import:

`tentacles.html`:
```html
<script>
define(function() {
  return /* module contents */;
})
</script>
```

This module is available at a relative URL, as shown in the above `squidbits`
example. There can be only one anonymous module per HTML document.

If you want to use an existing anonymous module (say, something defined
following the [UMD pattern](https://github.com/umdjs/umd)), give it a name with
the `as` attribute on a `<script>` tag:

```html
<script src="thingo.js" as="thingy"></script>
```

## Future work

### HTML dependencies.

Imagine:

 `foo.html`:
 ```html
 <template id="bar">...</template>
 ```

You could import a document:
 `index.html`:
 ```javascript
 define(['foo.html'], function(foo) {
   // foo is a document
   let bar = foo.querySelector('#bar');
   document.appendChild(document.importNode(bar.content));
 })
 ```

Or a node directly:
 `index.html`:
 ```javascript
 define(['foo.html#bar'], function(bar) {
   // bar is a HTMLTemplate node!
   document.appendChild(document.importNode(bar.content));
 })
 ```

### ES6

There is a lot to explore with ES6 module loading and HTML Imports. The loading
and ordering semantics appear to be fully compatible, and given a configurable
enough loader we should be able to let ES6 modules import HTML nodes and
vice-versa.
