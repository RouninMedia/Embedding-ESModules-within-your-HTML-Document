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

The following custom-built approach, involving `<script type="module/embedded" id="example-id">` and the asynchronous function `parseEmbeddedModule()` makes this possible:

#### Step One: Add two or more `<script type="module/embedded" id="example-id">` scripts to the HTML Document

**What is `type="module/embedded"`?**

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

Following this model, embedded modules look like this:

 - `<script type="module/embedded" id="example-id">`

Like `module`, even though it looks vaguely like it might be one, `module/embedded` is also not a MIME type.

Instead, it's just an indication of what type of script we're dealing with.

Two further things to note at this point:

1. Fairly importantly, `<script type="module/embedded" id="example-id">` is unofficial and unrecognised by the browser. Not being recognised by the browser means the element's contents will remain as unparsed `plaintext`. Since the element's contents are not parsed, they cannot be executed, either.

2. `<script type="module/embedded" id="example-id">` **must** include an `id` attribute. If it doesn't have one, it cannot be parsed by the `parseEmbeddedModule()` function.

_______

#### Step Two: Activate any Embedded Module via `parseEmbeddedModule()`

Once the **HTML Document** contains one or more `<script type="module/embedded" id="example-id">` elements, each of them can be explicitly activated via the asynchronous `parseEmbeddedModule()` function:

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

#### Step Three: The Embedded Module's `exports` are now retrievable from the `embeddedModules` object

An **Embedded Module** only needs to be _activated_ by `parseEmbeddedModule()` once.

When the **Embedded Module** is first _activated_, two things will happen:
 1. the **Embedded Module**'s `exports` will be asynchronously imported (via dynamic `import()`) into the `embeddedModules` object;
 2. one, several or all `exports` will then be retrieved from the `embeddedModules` object. 

From that point onwards, whenever requested, the **Embedded Module**'s `exports` will be synchronously retrieved from the `embeddedModules` object, with no further `import()`-ing necessary.

Since `parseEmbeddedModule()` is asynchronous, we can either use it in conjunction with `.then()` or else we can use it with with `await`, within an `async` function.

**Example with `.then()`:**
```
<script type="module/embedded" id="testModule1">

  const displayParagraph_1 = (text) => {
    const paragraph = document.createElement('p');
    paragraph.textContent = `1: ${text}`;
    document.body.appendChild(paragraph);
  }

  const paragraph_1 = 'This is a paragraph export PLUS a function export';
  export {displayParagraph_1, paragraph_1};

</script>

<script type="module/embedded" id="testModule2">

  const displayParagraph_2 = (text) => {
    const paragraph = document.createElement('p');
    paragraph.textContent = `2: ${text}`;
    document.body.appendChild(paragraph);
  }

  const paragraph_2 = 'This is also a paragraph export PLUS a function export';
  // export default paragraph_2;
  export {displayParagraph_2, paragraph_2};

</script>


```

__________

(*) Though it's also true that, long before **ESModules** were officially introduced, unofficial module-like design patterns already existed:

 - **Module Pattern (2007):** http://web.archive.org/web/20140903025035/http://yuiblog.com/blog/2007/06/12/module-pattern/
 - **Revealing Module Pattern (2007):** https://christianheilmann.com/2007/08/22/again-with-the-module-pattern-reveal-something-to-the-world/
 - **Definitive Module Pattern (2014):** http://inniepress.blogspot.com/2014/07/definitive-module-pattern.html
