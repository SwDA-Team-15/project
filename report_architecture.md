## 1. Context diagram - C4 Level 1 (draft)

This diagram follows the **C4 Model Level 1 (System Context)** approach. It shows the Express.js application as a single black box and maps out *who* talks to it, *what* it does, and *what* it depends on without exposing any internal implementation details.

## Diagram

![Express.js System Context Diagram](./img/context-diagram.png)

---

## External Actors (Top Layer)

These are the users and clients that send requests **into** the Express.js application.

| Actor | Interaction |
|---|---|
| 📱 **Mobile App Users** | Send **REST API calls** with JSON payloads over HTTP |
| 🔑 **API Consumers** | Third-party services authenticating via **Bearer Tokens** |
| 🌐 **Web Browser Users** | Send **HTTP/HTTPS requests**  page loads, form submissions |
| ⚙️ **Admin / DevOps** | **Deploy and configure** the application, manage infrastructure |

Each labeled arrow describes the protocol or data format used. Express.js is versatile enough to serve all four actor types simultaneously  REST APIs for mobile, token-authenticated endpoints for integrations, HTML pages for browsers, and admin tooling for operations.

---

## System Boundary (Center)

The central box represents the **Express.js Application**  the Node.js web framework responsible for receiving all incoming HTTP requests, routing them through a middleware pipeline (body parsing, authentication, logging, validation), executing business logic in route handlers, and returning appropriate responses (JSON, HTML, redirects, or errors).

At this abstraction level, every internal module the router, request/response objects, view engine, and middleware stack  is deliberately collapsed into a single element. The context diagram answers *"what does the system interact with?"*, not *"how does it work internally?"*.

---

## External Systems (Bottom Layer)

These are the **downstream services and infrastructure** the application depends on to function.

| System | Examples | Arrow Label | Purpose |
|---|---|---|---|
| 🌐 **CDN / Proxy** | Nginx, Cloudflare | Proxied Requests | Reverse proxy, SSL termination, caching |
| 📊 **Monitoring** | Sentry, Datadog | Logs & Metrics | Error tracking and performance monitoring |
| 🗄️ **Database** | PostgreSQL, MongoDB | SQL / NoSQL Queries | Persistent data storage and retrieval |
| ☁️ **Cloud Storage** | S3, GCS | File Upload/Download | Binary asset storage (user uploads, media) |
| 📧 **Email Service** | SendGrid, SES | Send Notifications | Transactional emails (resets, confirmations) |
| 💳 **Payment Gateway** | Stripe | Process Payments | Charges, subscriptions, and refunds |
| 🔒 **Auth Provider** | OAuth, JWT | Token Validation | Identity verification and session management |

The database uses a **cylinder shape**  standard C4 notation for data stores while all other systems use rectangles. The arrow labels describe *what kind of data* flows to each dependency, not the technical protocol.

## 2. Container Level - C4 Level 2

### Tool used

The diagram below was produced with **Structurizr**, rendered natively in GitHub Markdown. No external tool or build step is needed — the diagram lives directly in the report file and is always in sync with the text.

---

### Diagram

![Express.js Container Diagram](img/ContainerDiagram-dark.png)
---

### Explanation of each container

**HTTP Client**  
Anything that sends an HTTP request to the server — a browser, a mobile app, a curl command, another service. It only talks to the Node.js HTTP server; it has no knowledge of Express at all.

**Node.js HTTP Server** *(external)*  
The built-in `http` module creates a TCP socket, accepts connections, and calls the Express app function with the raw `IncomingMessage` (req) and `ServerResponse` (res) objects. Express is plugged in simply as `http.createServer(app)`.

**Express Core** (`lib/express.js`, `lib/application.js`)  
This is the entry point of the framework. It creates the app object via the factory function, stores application-level settings (`app.set()`), registers template engines (`app.engine()`), and owns the top-level `app.use()` and routing methods. On each incoming request it triggers the request/response enrichment and hands off to the Router.

**Router & Middleware Engine** (`lib/router/`)  
This is the heart of the request pipeline. It holds an ordered stack of `Layer` objects — each one wrapping a middleware function or a route handler. When a request comes in, it walks the stack, matches the request path and method, and calls each matching handler in order. Calling `next()` advances to the next layer; not calling it stops the chain. Error-handling middleware (four-argument functions) is a special layer type reserved for propagating errors down the chain.

**Request / Response Enrichment** (`lib/request.js`, `lib/response.js`, `lib/middleware/init.js`)  
Node's raw `req` and `res` objects are intentionally minimal. This container is responsible for extending them at the start of every request by swapping their prototype chain. After enrichment, `req` gains helpers like `req.params`, `req.query`, `req.accepts()`, and `req.is()`; `res` gains `res.json()`, `res.send()`, `res.redirect()`, `res.cookie()`, and `res.render()`. This happens once per request with no extra object allocation.

**View / Template Engine** (`lib/view.js` + `app.engine()`)  
This container handles `res.render()` calls. It resolves the template file path, picks the registered engine for that file's extension, and delegates the actual rendering to it. The engine itself is external — Express only calls it via the agreed interface `fn(path, options, callback)`. One app can have multiple engines registered for different file extensions at the same time.

**Static File Middleware** (`serve-static` package)  
An optional middleware layer mounted with `app.use(express.static('public'))`. When a request path matches a file on the filesystem it reads and streams it directly, bypassing all route handlers. If no file matches, it calls `next()` and the request continues down the middleware stack.

**Template Library** *(external)*  
A third-party npm package such as `pug`, `ejs`, or `handlebars`. Express knows nothing about its internals — it just calls the registered function and waits for the callback.

**File System** *(external)*  
The operating system's file system, used both by the static middleware (to serve assets) and by the view engine (to read template files).

---

### Relationship with Clean Architecture

Clean Architecture organises code into concentric layers — from the outermost (frameworks and drivers) inward to use cases and entities — with the rule that dependencies only point inward and inner layers know nothing about outer ones.

Express does not enforce Clean Architecture, but it is strongly compatible with it, and the container diagram makes this visible.

The **Node.js HTTP Server** and **Express Core** sit at the outermost layer, exactly where Clean Architecture places frameworks and drivers. They deal with the raw HTTP protocol and infrastructure concerns. The **Router & Middleware Engine** acts as the interface adapter layer: it translates incoming HTTP calls into function invocations and routes them to the right handler — the same job that controllers and presenters do in Clean Architecture. The **User Application** container (the developer's own code) is where business logic would live, corresponding to the use-case and entity layers. Crucially, this code does not import Express internals directly; it just registers functions with `app.use()` and `app.get()`, which means the dependency points outward — toward Express — keeping the core business logic free of framework knowledge.

The **View / Template Engine** and **Static File Middleware** containers sit at the boundary between infrastructure and interface adapters, consistent with where I/O-related adapters appear in Clean Architecture.

The main difference is that Express does not *enforce* this structure. There is nothing stopping a developer from putting database calls directly inside a route handler, which would mix use-case logic with interface adapter concerns and break the Clean Architecture principle. Express provides the *shape* of the layering but leaves adherence entirely up to the developer.

---

## 3. Component Level - C4 Level 3

### Tool used

The component diagram was created using PlantUML with the C4-PlantUML library. This tool was selected because it supports the C4 model notation and allows diagrams to be stored as text-based files inside the project repository.

### Component Diagram

The following diagram zooms into the Express Core container, mainly represented by the `lib/` directory. It shows the main internal components of Express.js and their relationships with Node.js APIs, the external router package, middleware packages, and template engines.

![Express.js Component Diagram](img/component-diagram.png)

### Component explanation

The component diagram focuses on the Express Core container of Express.js. The selected boundary is the `lib/` directory because it contains the main runtime implementation of the framework. External packages and Node.js built-in modules are shown as external dependencies and are not decomposed further.

The main entry point is `express.js`, which creates an Express application instance through `createApplication()`. This component exposes the public API used by application developers and connects the application object with the request and response prototypes.

The `application.js` component acts as the central application orchestrator. It manages application settings, middleware registration, routing delegation, mounted applications, server startup through `listen()`, and view rendering through `render()`.

The `request.js` component adapts Node.js `IncomingMessage` by adding Express-specific helper methods and properties. These include methods and properties for headers, content negotiation, protocol information, IP resolution, and query handling.

The `response.js` component adapts Node.js `ServerResponse` by adding high-level response helpers such as sending text or JSON responses, redirects, files, cookies, and rendered views.

The `view.js` component is responsible for resolving template files and invoking the selected template engine. It separates template lookup and rendering from the main application object.

The `utils.js` component provides shared helper functions used by other components, especially `application.js` and `response.js`. These utilities include ETag configuration, query parser configuration, trust proxy configuration, charset handling, and content-related helper functions.

This component decomposition was chosen because these files represent the main internal responsibilities of Express.js and provide a clear view of how the framework creates applications, handles requests, produces responses, and delegates optional rendering behavior.


## 4. Architectural Characteristics (draft)

To evaluate the overall quality of the Express.js architecture, we analyze its primary architectural characteristics (Quality Attributes) and demonstrate how they are structurally supported by the framework's design. To ground this analysis in objective software engineering principles, we support our reasoning using software metrics, specifically **Component Coupling** (inter-component dependencies) and **Cohesion** (intra-component single-purpose focus).

### 4.1 Extensibility and Maintainability

Extensibility is the defining architectural characteristic of Express.js, allowing developers to add arbitrary features without modifying the framework's core.

- **Supporting Architecture:** This attribute is achieved through the architectural decoupling of the core and the utilization of the *Chain of Responsibility* pattern via the middleware stack. The core system does not hardcode security, parsing, or logging mechanisms. Instead, it exposes a uniform pipeline where execution is delegated to independent middleware components.
- **Metrics-Based Evidence:** The system exhibits **High Functional Cohesion**. For instance, `lib/view.js` and `lib/utils.js` possess an internal **Fan-out of 0** relative to other core modules. They are entirely isolated leaf nodes in the internal structural graph. This near-zero internal coupling ensures that the template engine or utility sub-systems can be maintained, refactored, or replaced independently without triggering unexpected ripple effects (low change propagation probability) across the application hub.

### 4.2 Modularity and Interoperability (The Micro-Package Philosophy)

Express.js prioritizes a lightweight core footprint by shifting computational complexity to the outer edges of its ecosystem.

- **Supporting Architecture:** Instead of building a monolithic web framework, the architecture heavily relies on delegating atomic responsibilities to specialized, standalone external npm packages (e.g., `parseurl`, `send`, `accepts`).
- **Metrics-Based Evidence:** This strategy introduces an interesting architectural trade-off visible in the coupling metrics. Modules like `lib/response.js` present a very high **external Fan-out (16)**, connecting to a wide array of single-purpose external utilities. While high coupling is traditionally a risk factor, here it signifies a deliberate design decision: Express delegates specialized protocol specifications (like mime-types, cookie signing, or content disposition) to the open-source community, keeping the core codebase highly cohesive and focused purely on routing and delegation HTTP contracts.

### 4.3 Performance and Low Overhead

As a foundational web framework, minimizing runtime performance overhead and memory latency is critical for high-throughput network applications.

- **Supporting Architecture:** The internal architecture avoids heavy abstractions, deep inheritance layers, or complex runtime dependency injection containers. Objects are structured as thin wrappers that directly extend Node.js native streams (`http.IncomingMessage` and `http.ServerResponse`).
- **Metrics-Based Evidence:** This is reflected in the **Fan-in metrics** of the request and response modules. They act as fundamental abstractions heavily utilized by the orchestrators (`lib/express.js` and `lib/application.js`). By relying on efficient runtime prototype mutation (`merge-descriptors` and prototype reassignment inside `lib/middleware/init.js`) rather than creating deep encapsulation layers or factory proxies, the architecture ensures that wrapping a raw HTTP network stream into an Express context happens with near-zero memory and execution overhead.

### 4.4 Architectural Characteristics Summary

| Quality Attribute | Architectural Mechanism | Supporting Coupling/Cohesion Metric |
| :--- | :--- | :--- |
| **Extensibility** | Middleware Pipeline (Chain of Responsibility) | High Functional Cohesion; internal Fan-out of 0 for leaf modules (`view.js`). |
| **Maintainability**| Facade Isolation (`lib/express.js`) | Low Change Propagation; entry point is isolated from core internal refactoring. |
| **Modularity** | Micro-package delegation model | High external Fan-out (16 in `response.js`) balancing low core complexity. |
| **Performance** | Direct Node.js prototype extension | Low abstraction depth; high structural Fan-in for core HTTP wrappers. |
