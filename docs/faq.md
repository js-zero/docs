# FAQ

## How does JS Zero differ from TypeScript?

[TypeScript](http://www.typescriptlang.org/) aims to be a **superset** of JavaScript, while JS Zero aims to be a **subset**. This means TypeScript *extends* the JS language, while JS Zero *refines* the language by removing features that are hard to check for correctness.

## How does JS Zero differ from Flowtype?

[Flowtype](http://flowtype.org/) and JS Zero are indeed very similar. However, Flowtype aims to type check the **entire JavaScript language**. Unfortunately, JavaScript as a whole is too dynamic for Flowtype to achieve full type safety. JS Zero aims to specify a **subset** of JavaScript that can be type checked **soundly**.
