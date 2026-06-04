## 1. Context diagram (draft)

This diagram follows the **C4 Model Level 1 (System Context)** approach. It shows the Express.js application as a single black box and maps out *who* talks to it, *what* it does, and *what* it depends on without exposing any internal implementation details.

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


