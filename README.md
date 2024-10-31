# Embedding ESModules within your HTML Document

First introduced in 2015, **ESModules** allow JavaScript to take advantage of _modular script architecture_*.

From within an HTML document, you can call an external **ESModule** via `<script src="/scripts/my-module.js" type="module">`

From within an **ESModule** you may `import` another **ESModule** via two methods:

 - static `import`:
 - dynamic `import()`:

You can even embed an **ESModule** directly into an HTML document, via `<script type="module">`.

_However_ what you can't do - at least not natively - is embed multiple **ESModule** scripts into the same HTML document and then have those **ESModules** `import` and `export` from each other, resulting in a _modular script architecture_ which runs entirely within a _same-document-environment_.


__________

(*) Though it's also true that, long before **ESModules** were officially introduced, unofficial module architectures already existed:

 -
 -
