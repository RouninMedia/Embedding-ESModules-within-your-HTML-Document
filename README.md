# Embedding multiple ESModules within the same HTML Document

First introduced in 2015, **ESModules** enable JavaScript to be written using _modular script architecture_ (*).

### From HTML
From within an HTML document, you can call an external **ESModule** via `<script src="/scripts/my-module.js" type="module">`

### From an ESModule
From an **ESModule** - either an external `.js` file or one embedded in an HTML Document - you may `import` another **ESModule** via two methods:

 - static `import`: e.g. `import {export1 as alias1, export2 as alias2, /* … */} from '/module-name.js';`
 - dynamic `import()`: e.g. `let alias1, alias2; import('/my-module.js').then((myModule) => {alias1 = myModule.export1; alias2 = myModule.export2;});`

### From a Script
You may also `import()` an **ESModule** from a (non-module) JavaScript - either an external `.js` file or one embedded in an HTML Document - but _only_ via dynamic `import()`.

______

## However...

_However_ what you can't do - at least not natively - is embed _multiple_ **ESModule** scripts into the same HTML document and then have all those **ESModules** `import` and `export` from each other, resulting in a _modular script architecture_ which runs entirely within a _same-document-environment_.

The following custom-built approach, involving `<script type="module/embedded">` and the asynchronous function `parseEmbeddedModule()` makes this possible:

#### Step One: Add two or more `<script type="module/embedded">` scripts to the HTML Document

**What is `module/embedded`?**

**TLDR:** _It's unofficial._

**Longer explanation:** A long time ago, it was standard to include MIME type attributes in some elements:
 - `<script type="text/javascript">`
 - `<style type="text/css">`

As time progressed, these attributes went from _required_ (in **HTML 4.01**) to _optional_ (in **HTML5**) which led to their falling out of use:
 - `<script>`
 - `<style>`

In 2015, the `type` attribute made a comeback, this time to distinguish **ESModules** from conventional **JavaScript**:
 - `<script type="module">`

But, importantly, unlike the values from a decade earlier, the value `module` is _not_ a MIME type.

Instead, it's just an indication of what type of script we're dealing with.

Following this model, embedded modules have `module/embedded` as their `type`.

Like `module`, even though it looks vaguely like it might be one, `module/embedded` is also not a MIME type.

Instead, it's just an indication of what type of script we're dealing with.

Two further things to note at this point:

1. This is crucial: `<script type="module/embedded">` is unofficial and unrecognised by the browser. The element's contents will remain as `plaintext`. The element's contents will not be parsed by the browser and will not be executed.

2. This setup **requires** that a `<script type="module/embedded">` element includes an `id` attribute. If it doesn't have one, it cannot be parsed by the `parseEmbeddedModule()` function.

_______

#### Step Two: Activate any Embedded Module via `parseEmbeddedModule()`

Once the **HTML Document** contains one or more `<script type="module/embedded">` elements, each of them can be explicitly activated via the asynchronous `parseEmbeddedModule()` function:

##### `parseEmbeddedModule()` function
```js
const embeddedModules = {};

const parseEmbeddedModule = async (moduleName, exportNames = []) => {
  const embeddedModule = document.querySelector(`#${moduleName}`);
  if (embeddedModule?.getAttribute('type') !== 'module/embedded') throw new Error(`<script type="module/embedded" id="${moduleName}"> not found in document`);
  if (!embeddedModules[moduleName]) {
    embeddedModules[moduleName] = await import(URL.createObjectURL(new Blob([document.querySelector(`#${moduleName}`).textContent], {type: 'application/javascript'})))
  }
  return (typeof exportNames === 'string') 
    ? {[exportNames]: embeddedModules[moduleName][exportNames]}
    : (exportNames.length > 0) 
        ? Object.fromEntries(Object.entries(embeddedModules[moduleName]).filter(([key, value]) => exportNames.includes(key)))
        : embeddedModules[moduleName];
}
```
The function anticipates the following parameters:

 - a `moduleName` _(required)_
 - a list of one or more `exportNames` _(optional)_

If the function cannot find the **Embedded Module** by `id` in the same **HTML Document** it throws an error:

> `<script type="module/embedded" id="${moduleName}"> not found in document`

If the function can find the **Embedded Module**, it will check to see if the `exports` of that **Embedded Module** have already been imported into the `embeddedModules` object and, if not, it will _activate_ the **Embedded Module** and `import` them.

Once the `embeddedModules` object contains the **Embedded Module**'s `exports`, the function will return one, several or all of the exports, according to the `exportNames` parameter.

After returning one or more  **Embedded Module** `exports`, the function ends.

_______

#### Step Three: From now on, the Embedded Module's `exports` may be synchronously retrieved from the `embeddedModules` object

An **Embedded Module** only needs to be activated by `parseEmbeddedModule()` once.

Once the **Embedded Module** is activated, its `exports` will, from then on, be synchronously available from the `embeddedModules` object.



__________

(*) Though it's also true that, long before **ESModules** were officially introduced, unofficial module architectures already existed:

 -
 -
