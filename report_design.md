# Report Software Design

## 1. Dependencies

### 1.1 Methodology and Tools

To analyze the internal structure of Express.js, we focused on the core logic located in the `lib/` directory. We used **dependency-cruiser** to extract static code dependencies (`require` statements) and analyze structural coupling.

In addition to static dependencies, we also analysed how files co-evolve over time. To do this, we mined the Git history of the Express.js repository using `git log`, collected the set of files touched in each commit, and counted how often pairs of files appear together in the same commit (co-change frequency). We then focused on co-change pairs involving the core `lib/` modules identified above, so that we could directly compare static dependencies with these knowledge dependencies.

### 1.2 Code Dependencies (Results)

The analysis of the `lib/` directory revealed the following dependency metrics (Fan-in and Fan-out):

| Module | Fan-in | Fan-out | Architectural Role |
| :--- | :---: | :---: | :--- |
| `lib/express.js` | 0 | 8 | Entry Point |
| `lib/application.js` | 1 | 8 | Application Orchestrator |
| `lib/request.js` | 1 | 8 | HTTP Request Extender |
| `lib/response.js` | 1 | 16 | HTTP Response Extender |
| `lib/utils.js` | 2 | 8 | Internal Utilities |
| `lib/view.js` | 1 | 3 | Template View Renderer |

![Express.js Core Dependencies](img/dependency_graph.png)

### 1.3 Findings and Structural Analysis

Based on the full dependency graph and the coupling metrics extracted via dependency-cruiser, several key architectural and design characteristics of Express.js can be identified:

#### 1. Architectural Roles and Orchestration

The system demonstrates a clear top-down hierarchical flow of control:

- **`lib/express.js` (Fan-in: 0, Fan-out: 8):** Acts as a classic **Facade pattern** and the main entry point. It has no internal incoming dependencies, isolating the framework's core from the external global scope, while its high Fan-out reflects its responsibility to assemble and export the factory functions.
- **`lib/application.js` (Fan-in: 1, Fan-out: 8):** Serves as the central **Orchestrator**. It manages the server's lifecycle, application settings, and the integration of the router, which explains its high outgoing coupling to both internal prototypes and core Node.js subsystems (like `events` and `http`).

#### 2. The Heavy Weight of HTTP Protocol Extensions

The most notable structural asymmetry lies in the request and response abstractions:

- **`lib/request.js` (Fan-in: 1, Fan-out: 8) & `lib/response.js` (Fan-in: 1, Fan-out: 16):** These modules extend Node.js native `http.IncomingMessage` and `http.ServerResponse` prototypes.
- The massive Fan-out of `response.js` (16) makes it the most complex and "heavy" component in the architecture. This high coupling is justified by its design requirements: a modern web response wrapper must natively support dozens of distinct formatting operations, header manipulations, file streaming (`send`), and view template rendering (`view.js`).

#### 3. Adherence to Low Coupling and the Node.js Philosophy

- **`lib/view.js` (Fan-out: 3) and `lib/utils.js` (Fan-out: 8):** These represent the leaf nodes of the internal dependency tree. Their minimal coupling demonstrates a strict application of the **Low Coupling** principle. They remain highly stable and can be tested or refactored independently.
- **Delegation over Reinvention:** The overall analysis highlights a fundamental philosophy of Express.js: instead of creating a monolith, the framework maintains an extremely thin core. For atomic, specialized tasks (such as parsing cookies, determining mime-types, or evaluating proxy addresses), the core modules delegate execution to small, independent npm micro-packages. This keeps the framework lightweight while reusing proven community solutions.

### 1.4 Knowledge Dependencies

#### 1.4.1 Methodology

So far, we have looked at how modules depend on each other in the source code (via `require`), but this only tells part of the story. To make the evolution dimension explicit, we used the co-change data described in Section 1.1: for each commit we looked at the `.js` files modified together and computed how often each pair of files co-changes over the history. For every commit that touched at least two `.js` files, we built all file pairs and increased their co-change counter, and in the end, we sorted these pairs by how often they co-change.

We limited our analysis to pairs with relatively high co-change counts to avoid noise from occasional commits.

We then focused on pairs that involve the same core modules analysed in the previous subsections (for example, `lib/application.js`, `lib/request.js`, `lib/response.js`, `lib/utils.js`, `lib/view.js` and the router files). For each of the most frequent pairs, we checked whether there is also a static dependency between the two files, using the dependency-cruiser results and the dependency matrix that our teammate built for the core `lib/` components.

#### 1.4.2 Co-change Results

Table 2 shows a selection of the pairs that co-change most frequently and that involve at least one core module. The `co-change count` column tells how many commits modify both files together, and the last column indicates whether there is also a direct static dependency between them.

| File A               | File B                 | Co-change count | Code dependency? |
|----------------------|------------------------|-----------------|------------------|
| `lib/application.js` | `lib/response.js`      | 32              | yes              |
| `lib/response.js`    | `lib/utils.js`         | 30              | yes              |
| `lib/router/index.js`| `lib/router/route.js`  | 29              | yes              |
| `lib/request.js`     | `lib/response.js`      | 28              | yes              |
| `lib/application.js` | `lib/utils.js`         | 21              | yes              |
| `lib/application.js` | `lib/router/index.js`  | 21              | yes              |
| `lib/application.js` | `lib/express.js`       | 17              | yes              |
| `lib/express.js`     | `lib/response.js`      | 15              | yes              |
| `lib/application.js` | `lib/request.js`       | 13              | yes              |
| `lib/express.js`     | `lib/request.js`       | 13              | yes              |
| `lib/router/index.js`| `lib/utils.js`         | 12              | yes              |
| `lib/view.js`        | `test/view.test.js`    | 22              | no               |
| `lib/response.js`    | `test/res.send.js`     | 33              | no               |

Looking at these pairs, we see that the modules that are central in the static analysis (`application.js`, `request.js`, `response.js`, `utils.js`) also tend to be edited together quite often. This suggests that they behave as a tightly connected cluster: when the behaviour of requests and responses changes, the application orchestrator and utility functions are usually updated in the same commit to keep everything consistent. The router subsystem behaves similarly: `lib/router/index.js` and `lib/router/route.js` appear together in many commits, which fits with their shared responsibility for routing logic and the static dependencies highlighted in the matrix.

#### 1.4.3 Consistency and Inconsistency with Code Dependencies

For most core–core pairs in Table 2, the co-change behaviour is **consistent** with the static dependencies seen earlier. For example, `lib/application.js` and `lib/response.js` share 32 co-changes and are also directly connected in the dependency graph: the application object configures and uses the response prototype, so when new response helpers are added or existing behaviour is refactored, the application layer often needs to be adjusted accordingly. The same reasoning applies to `lib/request.js` and `lib/response.js`, which both extend the Node.js HTTP primitives; they form the main API surface for HTTP handling, so it is reasonable that they evolve together.

More interesting are the pairs where the relationship is **not visible** in the static graph. In particular, `lib/view.js`–`test/view.test.js` and `lib/response.js`–`test/res.send.js` have relatively high co-change counts but no direct code dependency from the core modules to the test files. These pairs capture a different kind of coupling: whenever the view rendering or response sending behaviour changes, the corresponding tests are updated in the same commit, even though tests do not appear in the dependency-cruiser matrix. This shows that a significant part of the system’s design knowledge lives in the test suite: the tests act as a contract for the behaviour of the core modules and evolve in sync with them, which is important to keep in mind when reasoning about future changes and their impact.

### 2 Patterns
