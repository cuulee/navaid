<div align="center">
  <img src="navaid.png" alt="navaid" height="160" />
</div>

<div align="center">
  <a href="https://npmjs.org/package/navaid">
    <img src="https://badgen.now.sh/npm/v/navaid" alt="version" />
  </a>
  <a href="https://travis-ci.org/lukeed/navaid">
    <img src="https://badgen.now.sh/travis/lukeed/navaid" alt="travis" />
  </a>
  <a href="https://npmjs.org/package/navaid">
    <img src="https://badgen.now.sh/npm/dm/navaid" alt="downloads" />
  </a>
  <a href="https://packagephobia.now.sh/result?p=navaid">
    <img src="https://packagephobia.now.sh/badge?p=navaid" alt="install size" />
  </a>
</div>

<div align="center">A navigation aid (aka, router) for the browser in 850 bytes~!</div>

## Install

```
$ npm install --save navaid
```


## Usage

```js
const Navaid = require('navaid');

// Setup router
let router = new Navaid();

// Attach routes
router
  .on('/', () => {
    console.log('~> /');
  })
  .on('/users/:username', params => {
    console.log('~> /users/:username', params);
  })
  .on('/books/*', params => {
    console.log('~> /books/*', params);
  });

// Run as single instance
router.run('/');
//=> "~> /"
router.run('/users/lukeed');
//=> "~> /users/:username { username: 'lukeed' }"
router.run('/books/kids/narnia');
//=> "~> /books/* { wild: 'kids/narnia' }"

// Run as long-lived router w/ history & "<a>" bindings
// Also immediately `.run()`s handler for current location
router.listen();
```

## API

### Navaid(options)

Returns: `Navaid`

#### options.base

Type: `String`<br>
Default: `'/'`

The base pathname for your application. By default, Navaid assumes it's operating on the root.

All routes will be processed _relative to_ this path (see [`on()`](#onpattern-handler)). Similarly, while [`listen`](#listen)ing, Navaid **will not** intercept any links that don't lead with this `base` pathname.

> **Note:** Navaid will accept a `base` with or without a leading and/or trailing slash, as all cases are handled identically.


### format(uri)
Returns: `String` or `false`

Formats and returns a pathname relative to the [`options.base`](#optionsbase) path.

If the `uri` **does not** begin with the `options.base`, then `false` will be returned instead. Otherwise, the return value will always lead with a slash (`/`).

> **Note:** This is called automatically within the [`listen()`](#listen) and [`run()`](#runuri) methods.

#### uri
Type: `String`

The path to format.


### route(uri, replace)
Returns: `undefined`

Programmatically route to a path whilst modifying & using the `history` events.

#### uri
Type: `String`

The desired path to navigate.

> **Important:** Navaid will prefix your `uri` with the [`options.base`](#optionsbase), if/when defined.

#### replace
Type: `Boolean`<br>
Default: `false`

If `true`, then [`history.replaceState`](https://developer.mozilla.org/en-US/docs/Web/API/History_API#The_replaceState()_method) will be used. Otherwise [`history.pushState`](https://developer.mozilla.org/en-US/docs/Web/API/History_API#Adding_and_modifying_history_entries) will be used.

This is generally used for redirects; for example, redirecting a user away from a login page if they're already logged in.


### on(pattern, handler)
Returns: `Navaid`

Define a function `handler` to run when a route matches the given `pattern` string.

#### pattern
Type: `String`

The supported pattern types are:

* static (`/users`)
* named parameters (`/users/:id`)
* nested parameters (`/users/:id/books/:title`)
* optional parameters (`/users/:id?/books/:title?`)
* any match / wildcards (`/users/*`)

> **Note:** It does not matter if your pattern begins with a `/` — it will be added if missing.


#### handler
Type: `Function`

The function to run when its `pattern` is matched.

> **Note:** This can also be a `Promise` or `async` function, but it's up to you to handle its result/error.

For `pattern`s with named parameters and/or wildcards, the `handler` will receive an "params" object containing the parsed values.

When wildcards are used, the "params" object will fill a `wild` key with the URL segment(s) that match from that point & onward.

```js
let r = new Navaid();

r.on('/users/:id', console.log).run('/users/lukeed');
//=> { id: 'lukeed' }

r.on('/users/:id/books/:title?', console.log).run('/users/lukeed/books/narnia');
//=> { id: 'lukeed', title: 'narnia' }

r.on('/users/:id/jobs/*').run('/users/lukeed/jobs/foo/bar/baz/bat');
//=> { id: 'lukeed', wild: 'foo/bar/baz/bat' }
```



### run(uri)
Returns: `Navaid`

Executes the matching handler for the specified path.

Unlike `route()`, this does not pass through the `history` state nor emit any events.

> **Note:** You'll generally want to use `listen()` instead, but `run()` may be useful in Node.js/SSR contexts.

#### uri
Type: `String`<br>
Default: `location.pathname`

The pathname to process. Its matching `handler` (as defined by [`on()`](#onpattern-handler)) will be executed.

### listen()
Returns: `Navaid`

Attaches global listeners to synchronize your router with URL changes, which allows Navaid to respond consitently to your browser's <kbd>BACK</kbd> and <kbd>FORWARD</kbd> buttons.

These are the (global) events that Navaid creates and/or responds to:

* popstate
* replacestate
* pushstate

Navaid will also bind to any `click` event(s) on anchor tags (`<a href="" />`) so long as the link has a valid `href` that matches the [`base`](#optionsbase) path. Navaid **will not** intercept links that have _any_ `target` attribute or if the link was clicked with a special modifier (eg; <kbd>ALT</kbd>, <kbd>SHIFT</kbd>, <kbd>CMD</kbd>, or <kbd>CTRL</kbd>).

### unlisten()
Returns: `undefined`

Detach all listeners initialized by [`listen()`](#listen).

> **Note:** This method is only available after `listen()` has been invoked.


## License

MIT © [Luke Edwards](https://lukeed.com)
