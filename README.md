# JS Zero - Zero bugs, Zero hassle.

**JS Zero** is a **ES6-compatible, proper subset** of JavaScript that aims to be functional and type-safe for doing I/O in web applications.

```text

  +--------------+        +---------------+        +--------------+
  |  Input       |        |  JS Zero      |        |  Output      |
  |  (HTTP)      |------> |  (Valid ES6)  |------> |  (JSON API)  |
  |  (Database)  |        |  (type safe)  |        |  (HTML)      |
  +--------------+        +---------------+        +--------------+

```

Read the [rest of the documentation and intro here](http://js-zero.com).

## Contributing

JS Zero is currently in its **design phase**. We're employing [Readme-Driven Development](http://tom.preston-werner.com/2010/08/23/readme-driven-development.html).

There are three ways you can contribute:

- [Discuss](http://discuss.js-zero.com) - help decide what exactly JS Zero should do, and techniques on how to do it!
- Maintain - help keep the docs (this repo) up to date with the latest [discussions](http://discuss.js-zero.com).
- Contribute (not yet) - once [the spec](http://js-zero.com/spec) is closer to being final, help write code that web developers will use everywhere.

### Building the Docs

You don't have to build the docs to contribute – you only need to submit a PR for something in the `docs/` folder. This is mostly for the doc website publisher.

#### Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs help` - Print this help message.
