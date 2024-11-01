# Embedding multiple ESModules within the same HTML Document

First introduced in 2015, **ESModules** enable JavaScript to be written using _modular script architecture_ (*).

## From HTML
From within an HTML document, you can call an external **ESModule** via `<script src="/scripts/my-module.js" type="module">`

## From an ESModule
From an **ESModule** - either in an external `.js` file or embedded in an HTML Document - you may `import` another **ESModule** via two methods:

 - static `import`: e.g. `import { export1 as alias1, export2 as alias2, /* … */ } from '/module-name.js';`
 - dynamic `import()`: e.g. `let alias1, alias2; import('/my-module.js').then((myModule) => {alias1 = myModule.export1; alias2 = myModule.export2;});

## From a Script
From a (non-module) JavaScript - either in an external `.js` file or embedded in an HTML Document - you may still `import()` an **ESModule**, but only via dynamic `import()`


_However_ what you can't do - at least not natively - is embed _multiple_ **ESModule** scripts into the same HTML document and then have those **ESModules** `import` and `export` from each other, resulting in a _modular script architecture_ which runs entirely within a _same-document-environment_.


__________

(*) Though it's also true that, long before **ESModules** were officially introduced, unofficial module architectures already existed:

 -
 -
