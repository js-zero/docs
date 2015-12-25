# FAQ

## How does JS Zero differ from TypeScript?

[TypeScript](http://www.typescriptlang.org/) aims to be a **superset** of JavaScript, while JS Zero aims to be a **subset**. This means TypeScript *extends* the JS language, while JS Zero *refines* the language by removing features that are hard to check for correctness.

## How does JS Zero differ from Flowtype?

[Flowtype](http://flowtype.org/) and JS Zero are indeed very similar. However, Flowtype aims to type check the **entire JavaScript language**. Unfortunately, JavaScript as a whole is too dynamic for Flowtype to achieve full type safety. JS Zero aims to specify a **subset** of JavaScript that can be type checked **soundly**.

Flow describes itself as `opt-in`; it doesn't check your code until you tell it to. To contrast, JS Zero is `opt-out`; it tries to check everything, but it's easy to tell it not to. Not only does this better allow for sound typing, but it also makes it easy to write and annotate your own dynamic functions that integrate seamlessly with the rest of the type system.
