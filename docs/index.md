![JS Zero](img/logo-medium.png)

# Zero bugs, Zero hassle.

*Current Status: [alpha prototype](https://github.com/js-zero/type-checker). We need your input!*

JavaScript is getting more and more popular every day. However, the dynamic aspect of JavaScript often makes it difficult to correctly write and maintain complex applications.

Many efforts have been made to create new, safer languages that compile *to* JavaScript. While many of these languages are designed very well, the fact is **most people already know JavaScript**, and are not so inclined to learn a new language to do the same things JavaScript can do.

**JS Zero** is a **ES6-compatible, proper subset** of JavaScript that aims to be functional and type-safe for doing I/O in web applications.

## Why JS Zero?

```text

  +--------------+        +---------------+        +--------------+
  |  Input       |        |  JS Zero      |        |  Output      |
  |  (HTTP)      |------> |  (Valid ES6)  |------> |  (JSON API)  |
  |  (Database)  |        |  (type safe)  |        |  (HTML)      |
  +--------------+        +---------------+        +--------------+

```

Much of programming in web development today is **"data in, data out"**. That is, given some data sources, we write programs to combine / aggregate / transform that data, then output the result.

JavaScript Zero fills this niche; it tries to make writing such programs bug-free, using a **typesafe, compatible subset** of the ubiquitous language of the web.

## Vision & Goals


JS Zero has three main goals:

- Specify a stricter, ES6-compatible, proper subset of JavaScript
- Design and build a type system that can type check this subset soundly
- As a developer tool, remain easy and seamless to use and understand.

With these goals complete, JS Zero will help you significantly towards writing bug-free programs.


## Motivating Example

Let's pretend you're making an app that allows a user to enter their **GitHub username** and receive back the **total number of lines** written across their repositories. This will involve using [GitHub's API](https://developer.github.com/v3/).

At this point, JS Zero already provides **a key feature** to help us:

- We must **declare the types** of the data we will get from the GitHub API. This is a reasonable requirement, since there is no way JS Zero can infer what kind of data GitHub will return.

Because JS Zero must be valid JavaScript, we must declare these types **in comments**. Here is what type declarations for the above example would like (notice the `@` symbols):

```javascript
// Although GitHub returns much more data,
// we only need to declare the types of data we care about.
//
let Repo = Object.of({ id: Number, name: String, languages_url: String })
let Languages = Map.of(String, Number)
```

As mentioned, the type of an API request cannot be inferred. That's ok; we can tell JS Zero to **assume** the type of a function instead of type checking the function's body. This means JS Zero will **trust** that our type declaration is correct.

In our example, we tell JS Zero to *assume* specific types for our API calls:

```javascript
// Pretend `HTTP` is a library that returns a promise

$assume `fetchRepos : (String) => Promise( Array(Repo) )`

function fetchRepos (user) {
  return HTTP.get('http://api.github.com/users/' + user + '/repos')
}


$assume `fetchLangs : (Repo) => Promise( Array(Language) )`

function fetchLangs (repo) {
  // Be careful! The above `$assume` means no type checking
  // will be done for the body of this function.

  return HTTP.get(repo.languages_url).then(function(langs) {
    var keyValPairs = Object.keys(langs).map( k => [k, langs[k]] )
    return new Map(keyValPairs)
  })
}
```

Now that we've declared the types of our HTTP requests, we can **safely** make use of them:

```javascript
// Note how there are NO TYPE DECLARATIONS in the following code!

myFormElement.addEventListener('submit', function(e) {
  e.preventDefault();

  fetchRepo(this.githubUsername.value)

    .then(function(repos) {
      // This line doesn't type check - r.language_url is not a property!
      // var promises = repos.map(r => fetchLangs(r.language_url));

      var promises = repos.map(r => fetchLangs(r.languages_url));
      return Promise.all(promises);
    })

    .then(function(repoLanguages) {
      // At this point, `repoLanguages` is an Array of Languages.
      // In other words, it has the concrete type: Array( Map(String, Number) ).
      // If that slipped by you, no problem. JS Zero has got you covered.

      var totalLineCount = repoLanguages
                           .map(langs => langs.values().reduce(add))
                           .reduce(add)
      ;
      alert("You have written " + totalLineCount + " lines of code!");
    })
  ;
});

function add (x, y) { return x + y; }
```

As you can see, once you annotate your boundaries (HTTP in this case), no further type declarations are necessary. The idea is, you record the shape of your data once, and then safely forget about it.

## Integrating with Other JS Code

Although all JS Zero code is valid JavaScript code, not all JavaScript code is valid Zero code. If you want your code to be type safe, sometimes you will have to annotate 3rd party libraries.


As it turns out, `$assume` is an easy way to do this if you only need a function or two. For example, let's say you want to use the [marked](https://github.com/chjj/marked) npm package to render some markdown:

```javascript
$assume `marked : (String) => String`
var marked = require('marked')

console.log( marked('I am using __markdown__.') )
```

On the other hand, if you're importing a larger library, you might need to write annotations yourself (assuming someone else has not already). For example, if you want to use [React.js](http://facebook.github.io/react/index.html) to handle your views, the type declarations might look something like this:

```javascript
$module('react', function () {
  let VirtualElement = $newType()
  let Attrs = Option.of( Object )
  let Child = Union(String, VirtualElement)

  return {
    createElement: Function.multi(
      `(String)                  => VirtualElement`,
      `(String, Attrs)           => VirtualElement`,
      `(String, Attrs, ...Child) => VirtualElement`
    ),
    render: `(VirtualElement, DomElement) => Void`
  }
})
var React = require('react')

// All type safe!
React.render(
  React.createElement('h1', null, 'Hello, world!'),
  document.getElementById('.my-div')
)
```

Most of the JSZ methods and types (`$module`, `$newType`, etc.) don't do any runtime computation; they are only there to inform the type checker in a fully ES6-compatible manner.

## Interested?

JS Zero is only in its design phase. If want JS Zero to happen, here are some ways you can help:

- Contribute to or star the [GitHub repo](https://github.com/js-zero/type-checker),
- Help spread the word with a [retweet](https://twitter.com/mindeavor/status/642722237452152832), or a [tweet of your own](https://twitter.com/intent/tweet?text=Help%20build%20a%20safer%20JavaScript%20with%20JS%20Zero!%20http%3A%2F%2Fjs-zero.com%2F),
- **Show your support / interest** by making a quick post [to this thread](http://discuss.js-zero.com/t/show-your-support/20) (easy GitHub login)
- Or [join the detailed discussion](http://discuss.js-zero.com/) and influence the direction of the project!
