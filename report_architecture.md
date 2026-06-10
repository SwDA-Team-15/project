## 1. Context diagram - C4 Level 1 

The Express.js Context Diagram, modeled at C4 Level 1, provides a high-level architectural view of the Express.js web framework and its relationships with the surrounding actors and external systems. The diagram identifies two primary human actors the Application Developer and the HTTP Client / End User , alongside five external systems that Express.js depends on or integrates with. Together, these elements illustrate how Express.js sits at the center of a Node.js web application ecosystem, orchestrating HTTP communication, middleware processing, routing, and view rendering.

### Diagram

![Express.js System Context Diagram](./img/context-diagram.png)

### Actors

The **Application Developer** is the person who builds web applications using Express.js. They interact with the framework by calling `require('express')()` and leveraging the public API to create application instances, register middleware, define routes, and configure view engines. The developer does not interact with the external systems directly through Express; rather, Express abstracts those interactions behind a clean, minimal API surface.

The **HTTP Client / End User** represents any browser, mobile application, or API consumer that initiates HTTP requests. These requests — carrying JSON, HTML, or other payloads — flow into the Express.js framework, which processes them through its middleware pipeline and returns appropriate HTTP responses. This bidirectional exchange forms the core runtime interaction of any Express-based application.

### The Central System

At the heart of the diagram is the **Express.js Framework** itself, classified as the `«system»` under analysis. It is a Node.js web framework responsible for creating application instances, managing the HTTP request/response lifecycle, executing the middleware pipeline, performing routing, and rendering views. Express acts as the orchestrator that connects all surrounding components, delegating specialized responsibilities to external packages and Node.js built-in modules.

### External Systems

Express.js interacts with five external systems, each fulfilling a distinct role in the overall architecture:

| External System | Role | Interaction with Express.js |
|---|---|---|
| **Node.js Built-in APIs** | Core runtime platform providing HTTP, Events, File System, and Path modules | Express uses `http.createServer()` inside `app.listen()`, and relies on `fs`, `path`, and `events` for file operations and event handling |
| **Template Engines** | Server-side HTML rendering (e.g., EJS, Pug, Handlebars) | Express loads and invokes the engine's `render` function when `res.render()` is called, enabling dynamic view generation |
| **router package** | External routing and middleware stack used by Express 5 | Express exposes `Router` and `Route` objects backed by this package, enabling modular route definitions and nested middleware chains |
| **serve-static** | Static file-serving middleware | Exported by Express as `express.static()`, allowing applications to serve CSS, JavaScript, images, and other static assets from a directory |
| **body-parser** | Request body parsing middleware for JSON, raw, text, and URL-encoded data | Exported as middleware helpers (`express.json()`, `express.urlencoded()`), enabling automatic parsing of incoming request bodies |

---


## 2. Container Level - C4 Level 2

### Tool used

The diagram below was produced with **Structurizr**, rendered natively in GitHub Markdown.

---

### Diagram

![Express.js Container Diagram](img/ContainerDiagram-dark.png)

---

### Explanation of each container

**Application Developer** *(person)*  
The developer who builds the web application using Express. They use the framework's public API to register routes, add middleware, and configure the app.

**HTTP Client / End User** *(person)*  
Anyone who sends an HTTP request to the running app — a browser, a mobile app, or another service calling an API.

**Express Core** *(internal — `lib/express.js`, `lib/application.js`)*  
This is the starting point of the whole framework. When you call `express()`, this is what runs. It creates the application instance via `createApplication()`, keeps track of all the settings via `app.set()`, registers template engines via `app.engine()`, and exposes the API the developer uses to build their app. Every incoming request passes through here first before anything else happens.

**Router & Middleware Engine** *(internal — `lib/router/index.js`, `lib/router/layer.js`, `lib/router/route.js`)*  
This is where the actual request handling lives. It keeps an ordered list of middleware functions and route handlers registered via `app.use()` and `app.get/post/put/delete()`. When a request comes in, `router.handle()` goes through each one in order until something sends a response back. If a piece of middleware is done with its job, it just calls `next()` to move to the next one in line.

**Request / Response Enrichment** *(internal — `lib/request.js`, `lib/response.js`, `lib/middleware/init.js`)*  
Node.js gives Express a very basic request and response object. This container upgrades both of them at the start of every request using `Object.setPrototypeOf()`. After that, `req` gets helpers like `req.params`, `req.query`, and `req.accepts()`, and `res` gets things like `res.json()`, `res.send()`, `res.redirect()`, and `res.render()`.

**View Engine** *(internal — `lib/view.js`)*  
When a route handler calls `res.render()`, the request ends up here. `View.prototype.render()` figures out which template file to use and which rendering library to hand it off to, based on the file extension registered via `app.engine()`. It does not do any rendering itself — it just coordinates the handoff to the external template engine.

**Static File Middleware** *(internal — `lib/middleware/init.js`, wraps `serve-static`)*  
This container handles requests for static assets like images, CSS, and JavaScript files. It is mounted via `app.use(express.static('public'))`. If the requested file exists on the filesystem, it gets sent back directly without going through any route handler. If not, it calls `next()` and the request continues down the stack.

**Body Parser Middleware** *(internal — wraps `body-parser`, exposed via `lib/express.js`)*  
When a request comes in with a body — like a JSON payload or form data — this container reads and parses it so route handlers can access it via `req.body`. It is exposed by Express as `express.json()` and `express.urlencoded()`. Without this, the developer would have to read and parse the raw data stream themselves.

**Node.js Built-in APIs** *(external)*  
The core of Node.js itself. Express relies on `http.createServer()` inside `app.listen()` to actually create the server and accept connections, and also uses `fs`, `path`, and `events` for file operations and event handling.

**Template Engines** *(external)*  
Third-party libraries like EJS, Pug, or Handlebars that do the actual work of turning a template file and some data into an HTML page. Express supports any of them as long as they follow the `fn(path, options, callback)` interface registered via `app.engine()`.

**router package** *(external)*  
An external npm package that Express 5 uses under the hood to manage route matching and the middleware stack. The Router & Middleware Engine container is built on top of it, using its `Route` and `Layer` objects.

**serve-static** *(external)*  
The npm package that the Static File Middleware container wraps. It contains all the actual logic for finding and streaming files from the filesystem in response to a request.

**body-parser** *(external)*  
The npm package that the Body Parser Middleware container wraps. It handles the low-level parsing of request body formats including JSON, URL-encoded, raw, and plain text.

---

## 3. Component Level - C4 Level 3

### Tool used

The component diagram was created with **PlantUML** and the **C4-PlantUML** library.

### Component Diagram

This diagram zooms into the **Express Core** container defined in the previous C4 Level 2 diagram. The focus is on the internal responsibilities mainly implemented in `lib/express.js` and `lib/application.js`, while the other Level 2 containers are shown as external collaborators.

![Express.js Component Diagram](img/component-diagram.png)

### Component explanation

The **Express Core Container** is decomposed into components that represent its main runtime responsibilities.

The **Public API Facade** exposes the developer-facing API, including `express()`, `Router`, `Route`, and middleware helper functions. The **Application Factory** creates the callable `app` object through `createApplication()`, attaches prototypes, and initializes the application instance.

The **Settings & Engine Registry** stores application configuration through methods such as `app.set()`, `app.get()`, and `app.engine()`. The **Middleware & Route Registration API** handles `app.use()`, `app.route()`, and HTTP verb methods, then delegates registered handlers to the Router & Middleware Engine.

The **Request Lifecycle Coordinator** represents `app.handle()`. It receives requests from the Node.js runtime, prepares enriched request/response objects, and delegates execution to the middleware stack. The **Rendering Coordinator** represents `app.render()`, reading view settings and delegating rendering to the View Engine.

The **HTTP Server Bootstrapper** represents `app.listen()`, which uses `http.createServer()` to bind the Express application to the Node.js HTTP runtime. Finally, the **Middleware Export Adapter** exposes shortcuts such as `express.json()`, `express.urlencoded()`, and `express.static()`, while delegating the actual work to Body Parser and Static File Middleware.

This decomposition keeps the Level 3 diagram consistent with the Container Level view: Router, Request/Response Enrichment, View Engine, Static File Middleware, and Body Parser Middleware remain outside Express Core and interact with it as collaborating containers.
### SOLID Principles Analysis

The table below summarises how the main core modules in `lib/` relate to the SOLID principles at component level.

| Principle | Assessment | Components and Analysis |
|----------|------------|-------------------------|
| **Single Responsibility (SRP)** | Partially satisfied | • `express.js`: mostly focused on creating the app and exposing the public API, so the responsibility is clear. <br>• `view.js`: only deals with template lookup and calling the selected engine, which fits SRP well. <br>• `application.js`: mixes settings, middleware registration, routing, template engine setup, rendering and `app.listen()`, so it clearly does more than one job. <br>• `response.js`: one large module that handles status, headers, cookies, JSON, file streaming, redirects and view rendering. |
| **Open/Closed (OCP)** | Good at extension points, weaker inside core | • Middleware (`app.use()`): new behaviour is usually added by registering extra middleware without touching core files. <br>• Views (`app.engine()` + `view.js`): new template engines can be plugged in through a stable callback API. <br>• `application.js` and `response.js`: when new features are added to Express itself, these modules are often edited, so they are not really “closed for modification”. |
| **Liskov Substitution (LSP)** | Largely respected by design | • `request.js` and `response.js`: extend `http.IncomingMessage` and `http.ServerResponse` but keep their original behaviour, so they can still be used where plain Node objects are expected. <br>• Template engines: can be swapped as long as they follow the expected `render(path, options, callback)` style contract. |
| **Interface Segregation (ISP)** | Mixed outcome | • From the application side, each middleware only uses a small set of methods on `req`, `res` and the app. <br>• Internally, `application.js`, `request.js` and especially `response.js` expose wide APIs that cover many different concerns; there are no smaller, dedicated interfaces for configuration, routing or response types. |
| **Dependency Inversion (DIP)** | Applied at public boundaries, limited inside HTTP layer | • Middleware and strategies: Express depends on simple function contracts (`(req, res, next)`, ETag functions, query parsers, trust‑proxy functions), which users can provide without changing the framework. <br>• Template engines: `view.js` calls engines via a generic render function and does not depend on a specific library. <br>• Inside `request.js` and `response.js`, the code imports concrete packages like `send`, `cookie`, `accepts` and `type-is` directly. |

## 4. Architectural Characteristics

To evaluate the overall quality of the Express.js architecture, we analyze its primary architectural characteristics (Quality Attributes) and demonstrate how they are structurally supported by the framework's design. To ground this analysis in objective software engineering principles, we support our reasoning using software metrics, specifically **Component Coupling** (inter-component dependencies) and **Cohesion** (intra-component single-purpose focus).

### 4.1 Extensibility and Maintainability

Extensibility is the defining architectural characteristic of Express.js, allowing developers to add arbitrary features without modifying the framework's core.

- **Supporting Architecture:** This attribute is achieved through the architectural decoupling of the core and the utilization of the *Chain of Responsibility* pattern via the middleware stack. The core system does not hardcode security, parsing, or logging mechanisms. Instead, it exposes a uniform pipeline where execution is delegated to independent middleware components.
- **Metrics-Based Evidence:** The system exhibits **High Functional Cohesion**. For instance, `lib/view.js` and `lib/utils.js` possess an internal Fan-out of 0 relative to other core modules. They are entirely isolated leaf nodes in the internal structural graph. This near-zero internal coupling ensures that the template engine or utility sub-systems can be maintained, refactored, or replaced independently without triggering unexpected ripple effects (low change propagation probability) across the application hub.

### 4.2 Modularity and Interoperability (The Micro-Package Philosophy)

Express.js prioritizes a lightweight core footprint by shifting computational complexity to the outer edges of its ecosystem.

- **Supporting Architecture:** Instead of building a monolithic web framework, the architecture heavily relies on delegating atomic responsibilities to specialized, standalone external npm packages (for example, `parseurl`, `send`, `accepts`).
- **Metrics-Based Evidence:** This strategy introduces an interesting architectural trade-off visible in the coupling metrics. Modules like `lib/response.js` present a very high external Fan-out (16), connecting to a wide array of single-purpose external utilities. While this concentration of external dependencies causes a technical violation of the Single Responsibility Principle (SRP) at the component level, it signifies a deliberate design decision: Express delegates specialized protocol specifications (like mime-types, cookie signing, or content disposition) to the open-source community, keeping the core codebase highly cohesive and focused purely on routing and delegation HTTP contracts.

### 4.3 Performance and Low Overhead

As a foundational web framework, minimizing runtime performance overhead and memory latency is critical for high-throughput network applications.

- **Supporting Architecture:** The internal architecture avoids heavy abstractions, deep inheritance layers, or complex runtime dependency injection containers. Objects are structured as thin Runtime Adapters that directly mutate and enrich Node.js native streams (`http.IncomingMessage` and `http.ServerResponse`).
- **Metrics-Based Evidence:** This is reflected in the Fan-in metrics of the request and response modules. They act as fundamental abstractions heavily utilized by the orchestrators (`lib/express.js` and `lib/application.js`). By relying on efficient runtime prototype mutation (`merge-descriptors` and prototype reassignment inside `lib/middleware/init.js`) rather than creating deep encapsulation layers or factory proxies, the architecture ensures that wrapping a raw HTTP network stream into an Express context happens with near-zero memory allocation and minimal execution overhead.

### 4.4 Architectural Characteristics Summary

| Quality Attribute | Architectural Mechanism | Supporting Coupling/Cohesion Metric |
| :--- | :--- | :--- |
| **Extensibility** | Middleware Pipeline (Chain of Responsibility) | High Functional Cohesion; internal Fan-out of 0 for leaf modules (`view.js`). |
| **Maintainability**| Facade Isolation (`lib/express.js`) | Low Change Propagation; entry point is isolated from core internal refactoring. |
| **Modularity** | Micro-package delegation model | High external Fan-out (16 in `response.js`) balancing low core complexity and structural SRP trade-offs. |
| **Performance** | Runtime Prototype Adaptation | Low abstraction depth; high structural Fan-in for core HTTP wrappers. |
