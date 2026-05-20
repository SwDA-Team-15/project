# Design Patterns in Express.js

## Overview

Express.js is a small web framework for Node.js. When you look at its source code (inside the `/lib` folder on GitHub), you can spot at least four classic design patterns being used. Each one solves a specific problem that came up while building the framework.

| # | Pattern | File in Express | Problem it solves |
|---|---------|----------------|------------------|
| 1 | **Factory** | `lib/express.js` | How to create an app object the right way |
| 2 | **Chain of Responsibility** | `lib/router/index.js` | How requests flow through middleware |
| 3 | **Decorator** | `lib/middleware/init.js` | How to add new features to `req` and `res` |
| 4 | **Strategy** | `lib/view.js` + `app.engine()` | How to support different template engines |

---

## Pattern 1 — Factory

### What it is

A Factory is just a function that builds and returns a ready-to-use object. Instead of calling `new SomeClass()` yourself, you call a function and it handles all the setup for you.

### Where it is in Express

In `lib/express.js` there is a function called `createApplication`. This is the factory. Every time you do `const app = express()`, you are calling it.

```js
function createApplication() {
  var app = function(req, res, next) {
    app.handle(req, res, next);
  };

  mixin(app, EventEmitter.prototype, false);
  mixin(app, proto, false);

  app.request  = Object.create(req, { app: { value: app } });
  app.response = Object.create(res, { app: { value: app } });

  app.init();
  return app;
}
```

### Who plays which role?

| Role | What it is in Express |
|------|-----------------------|
| **Factory** | `createApplication()` in `lib/express.js` |
| **Product** | The `app` object that gets returned |
| **Client** | Your own code when you write `const app = express()` |

### Why is this pattern used here?

The problem is a bit tricky. Node.js expects you to pass a *function* to `http.createServer(app)`, but that same function also needs to have lots of methods on it like `app.use()`, `app.get()`, `app.listen()`, and so on. In JavaScript, you can't really do that cleanly with a normal class. The factory function solves it by creating a plain function and then mixing in all the extra methods afterwards, before returning it.

Without the factory, whoever is creating an app would need to manually set up the prototype chain, call `init()`, and wire everything together. That would be messy and easy to get wrong.

### Is there an alternative?

You could use an ES6 class instead.

```js
class Application extends EventEmitter { ... }
const app = new Application();
```

**Pros:** It is more modern and easier to read.

**Cons:** A class always returns a plain object when you use `new`. You can't `new` something into a *function*, which is exactly what Express needs for `http.createServer(app)` to work. So this approach would break one of the core features of the framework.

---

## Pattern 2 — Chain of Responsibility

### What it is

Chain of Responsibility is when you have a line of handlers, and a request gets passed down the line one by one. Each handler can either deal with the request and stop, or pass it along to the next one.

### Where it is in Express

This is basically the whole middleware system. Every time you call `app.use()` you are adding a new handler to the chain. When a request comes in, Express goes through each one in order.

```js
app.use(loggerMiddleware);   // first handler
app.use(authMiddleware);     // second handler
app.get('/home', homeHandler); // third handler

function loggerMiddleware(req, res, next) {
  console.log(req.method, req.url);
  next(); // pass to the next handler
}

function authMiddleware(req, res, next) {
  if (!req.headers['x-auth']) return next(new Error('Not allowed'));
  next();
}
```

Inside `lib/router/index.js`, Express stores all these handlers in an array called `stack`. When a request arrives, it loops through them one by one.

### Who plays which role?

| Role | What it is in Express |
|------|-----------------------|
| **Handler** | Any middleware function with `(req, res, next)` |
| **Chain** | The `stack` array inside the Router |
| **Chain manager** | `router.handle()` — the loop that runs each handler |
| **Request** | The `req` and `res` objects being passed around |

### Why is this pattern used here?

An HTTP server needs to do a lot of things with each request: log it, check if the user is logged in, parse the body, find the right route, and so on. The number of steps changes depending on the app. Chain of Responsibility lets each step be its own small independent function. You can add or remove steps without touching anything else. If one handler wants to stop the request (for example, return a 401), it just doesn't call `next()`.

Without this pattern, you would have to put all the logic for every possible request into one giant function, which would be a nightmare to maintain.

### Is there an alternative?

You could use Node.js events instead.

```js
app.on('request', logRequest);
app.on('request', authenticate);
```

**Pros:** It is very natural in Node.js and easy to add listeners.

**Cons:** Event listeners don't have a guaranteed order. There is also no clean way for one listener to stop the others, and passing errors between them is very awkward. The `next(err)` pattern in Express is much cleaner.

---

## Pattern 3 — Decorator

### What it is

Decorator is when you add new features or methods to an existing object *without* changing the original code. You wrap or extend the object at runtime.

### Where it is in Express

Node.js gives you a raw `req` (request) object and a raw `res` (response) object. These are very basic — they can barely do anything useful on their own. Express extends them by changing their prototype at the start of every request.

In `lib/middleware/init.js`:

```js
exports.init = function(app) {
  return function expressInit(req, res, next) {
    Object.setPrototypeOf(req, app.request);  // decorate req
    Object.setPrototypeOf(res, app.response); // decorate res
    res.locals = res.locals || Object.create(null);
    next();
  };
};
```

After this runs, the `req` object suddenly has things like `req.params`, `req.query`, and `req.accepts()`. The `res` object gets `res.json()`, `res.send()`, `res.render()`, `res.redirect()`, and more. All of these extra methods are defined in `lib/request.js` and `lib/response.js`.

### Who plays which role?

| Role | What it is in Express |
|------|-----------------------|
| **Original object** | Node's built-in `http.IncomingMessage` and `http.ServerResponse` |
| **Decorator** | `lib/request.js` and `lib/response.js` (the extended prototypes) |
| **Where decoration happens** | `lib/middleware/init.js` and `app.handle()` |
| **Client** | Your route handlers using `req.*` and `res.*` |

### Why is this pattern used here?

Express cannot change or subclass Node's built-in objects — they are created by Node *before* Express even sees them. The only option is to "decorate" them after the fact by swapping out their prototype. This is actually pretty clever because it adds all the useful methods with almost no overhead — no new objects are created, just the prototype link is updated.

Without this pattern, you would either have to create a wrapper object for every request (which wastes memory) or put all the helper methods directly on Node's global prototypes (which would pollute the whole runtime).

### Is there an alternative?

You could create a wrapper class.

```js
class ExpressRequest {
  constructor(nodeReq) { this._raw = nodeReq; }
  get url() { return this._raw.url; }
  accepts(type) { /* ... */ }
}
```

**Pros:** Very clean and explicit. Easy to understand what methods come from where.

**Cons:** You would have to manually proxy every single property and method from the original Node object. There are a lot of them. It also creates a new wrapper object for every single request, which adds memory pressure. And third-party libraries that expect a raw Node request object would break.

---

## Pattern 4 — Strategy

### What it is

Strategy is when you have multiple ways of doing the same thing, and you want to be able to swap between them without changing the rest of your code. You pick which "strategy" to use at runtime.

### Where it is in Express

This shows up in the template engine system. Express supports Pug, EJS, Handlebars, and many others. Each one is a separate rendering "strategy". You register one with `app.engine()` and Express calls it when you do `res.render()`.

```js
// Registering different strategies
app.engine('pug', require('pug').__express);
app.engine('ejs', require('ejs').renderFile);

app.set('view engine', 'pug'); // pick the default

// lib/application.js — stores each engine in a map
app.engine = function engine(ext, fn) {
  var extension = ext[0] !== '.' ? '.' + ext : ext;
  this.engines[extension] = fn; // fn is the strategy
  return this;
};
```

Then in `lib/view.js`, when Express actually needs to render something:

```js
View.prototype.render = function render(options, callback) {
  this.engine(this.path, options, callback); // calls whichever engine was registered
};
```

All template engines must follow the same contract: `fn(path, options, callback)`. As long as a library follows that signature, it works with Express.

### Who plays which role?

| Role | What it is in Express |
|------|-----------------------|
| **Context** | `View` class in `lib/view.js` and `app.render()` |
| **Strategy interface** | The function signature `(path, options, callback)` |
| **Concrete strategies** | `pug.__express`, `ejs.renderFile`, `hbs.__express`, etc. |
| **Strategy registry** | `this.engines` map on the app object |

### Why is this pattern used here?

Different teams like different template engines, and there is no single "best" one. Express does not want to force you to use Pug if you prefer EJS. With the Strategy pattern, Express core has zero engine-specific code — it just calls whatever function you registered. You can even use a completely custom engine that nobody else has heard of, as long as it follows the `(path, options, callback)` interface.

### Is there an alternative?

You could hardcode each engine with an if/else chain.

```js
if (engine === 'pug') pug.renderFile(path, options, cb);
else if (engine === 'ejs') ejs.renderFile(path, options, cb);
```

**Pros:** Very simple to implement at first.

**Cons:** Every time someone wants to add a new template engine, they would need to edit the Express source code itself. That makes the framework hard to extend and puts unnecessary maintenance pressure on the core team. The strategy pattern completely avoids this by letting anyone plug in their own engine from outside.

---

## Summary

| Pattern | Problem in Express | How it solves it |
|---------|--------------------|-----------------|
| **Factory** | Building an app object that is also a callable function | `createApplication()` mixes everything together before returning the app |
| **Chain of Responsibility** | Passing a request through an unknown number of middleware steps | An ordered `stack` of handlers; `next()` moves to the next one |
| **Decorator** | Adding useful methods to Node's bare `req`/`res` objects | Prototype is swapped at request time to include all Express methods |
| **Strategy** | Supporting many template engines without knowing them in advance | A registry of engine functions; any function with the right signature works |

What is interesting about Express is that these four patterns together explain why the framework is so small but so flexible. The factory sets things up cleanly, the chain handles any number of middleware steps, the decorator enriches the objects you work with, and the strategy keeps rendering completely open for extension. None of them are complicated on their own, but put together they make the whole framework work.
