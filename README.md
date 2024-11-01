# Embedding multiple ESModules within the same HTML Document

First introduced in 2015, **ESModules** enable JavaScript to be written using _modular script architecture_ (*).

### From HTML
From within an HTML document, you can call an external **ESModule** via `<script src="/scripts/my-module.js" type="module">`

### From an ESModule
From an **ESModule** - either an external `.js` file or one embedded in an HTML Document - you may `import` another **ESModule** via two methods:

 - static `import`: e.g. `import { export1 as alias1, export2 as alias2, /* … */ } from '/module-name.js';`
 - dynamic `import()`: e.g. `let alias1, alias2; import('/my-module.js').then((myModule) => {alias1 = myModule.export1; alias2 = myModule.export2;});

### From a Script
From a (non-module) JavaScript - either an external `.js` file or one embedded in an HTML Document - you may still `import()` an **ESModule**, but only via dynamic `import()`

______

## However...

_However_ what you can't do - at least not natively - is embed _multiple_ **ESModule** scripts into the same HTML document and then have all those **ESModules** `import` and `export` from each other, resulting in a _modular script architecture_ which runs entirely within a _same-document-environment_.

The following approach, involving `<script type="module/embedded">` and the asynchronous function `parseEmbeddedModule()` makes this possible.

#### Step One: Add two or more `<script type="module/embedded">` scripts to the HTML Document

A long time ago, it was standard to include MIME type attributes in some elements:
 - `<script type="text/javascript">`
 - `<style type="text/css">`

As time progressed, these attributes went from _required_ (in **HTML 4.01**) to _optional_ (in **HTML5**) which led to their falling out of use:
 - `<script>`
 - `<style>`

In 2015, the `type` attribute made a comeback, this time to distinguish **ESModules** from conventional **JavaScript**:
 - `<script type="module">`

Notably, `module` is _not_ a MIME type.

It's just an indication of what type of script we're dealing with.

Following this model, embedded modules have `module/embedded` as their `type`.

`module/embedded` is also not a MIME type (even though it looks vaguely like it might be one).

Instead it's just an indication of what type of script we're dealing with.

Two further things to note:

1. Quite importantly - because `<script type="module/embedded">` is unofficial and unrecognised by the browser, its contents will remain as `plaintext`. The contents will not be parsed by the browser and will not be executed.
2. It is **required** that a `<script type="module/embedded">` element includes an `id` attribute.

#### Step Two: Activate any Embedded Module via `parseEmbeddedModule()`

#### Step Three: Including at the moment of activation, once an Embedded Module is activated, any of its `exports` may be accessed of `parseEmbeddedModule()`


__________

(*) Though it's also true that, long before **ESModules** were officially introduced, unofficial module architectures already existed:

 -
 -
