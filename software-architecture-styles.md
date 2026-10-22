# Software Architecture Styles

An architecture style describes how an application is organized and how its
parts communicate. Styles are not always alternatives: for example, a
**modular monolith** can use **layered** organization and **Clean Architecture**
dependency rules at the same time.

## Common styles

### Monolith

One application is built, deployed, and usually run as a single unit. Its code
may be organized in any number of ways; "monolith" describes the deployment
unit, not necessarily a tangled codebase.

**Pros**

- Simple to develop, debug, test, and deploy when the system is small.
- In-process calls are straightforward and avoid network latency.
- A single deployment can make transactions across features simpler.

**Cons**

- As it grows, unclear boundaries can make changes risky and development
  harder to coordinate.
- The whole application is commonly deployed and scaled together.
- A defect or resource-heavy feature can affect the whole process.

**C# example:** one ASP.NET Core web application containing controllers,
services, and data access, published as a single deployment.

### Modular monolith

A single deployable application is divided into explicit modules, each owning
a business capability and keeping its implementation details private. Modules
communicate through deliberate interfaces rather than reaching into one
another's internals.

**Pros**

- Retains simple deployment and in-process communication.
- Clear boundaries make code easier to understand, test, and change.
- Can provide a path to independently deployed services later if there is a
  real need.

**Cons**

- Module boundaries require discipline; shared tables or direct access to
  another module's internals can undermine them.
- Modules cannot be scaled or deployed independently.
- Extracting a module into a service later still requires handling network
  failures, data ownership, and distributed operations.

**C# example:** one ASP.NET Core host with `Orders` and `Catalog` modules.
Keep each module's application logic and persistence behind its own public
interfaces; avoid a single shared "everything" service or unrestricted
cross-module `DbContext` access.

### Layered (n-tier)

Code is grouped by technical responsibility, often into presentation,
application/business logic, and data-access layers. In a traditional
**n-tier** system, those tiers may also run on separate machines or processes;
layers are a code organization, while tiers refer to deployment boundaries.

**Pros**

- Familiar structure with responsibilities that are easy to explain.
- Centralizes common concerns such as HTTP handling and persistence.
- Works well for many CRUD-oriented applications.

**Cons**

- A change to one feature may require edits across many layers.
- Layers can become pass-through wrappers that add complexity without value.
- If dependencies are allowed to flow freely inward from business logic to
  infrastructure, business rules become harder to test independently.

**C# example:** `Web` calls `Application`, which calls `Infrastructure` for
database access. Keep dependencies intentional; a layered layout alone does
not guarantee good separation.

### Clean Architecture / Hexagonal (Ports and Adapters) / Onion

These related approaches organize code around the domain and use dependency
inversion: core business rules define abstractions (ports), while outer
infrastructure components implement them (adapters). The names and diagrams
differ, but the shared goal is to keep business rules independent of UI,
databases, and frameworks.

**Pros**

- Domain and application logic can be tested without a live database or web
  server.
- Infrastructure can often be replaced with less impact on core rules.
- Makes dependencies and integration boundaries explicit.

**Cons**

- Adds interfaces, projects, and indirection that may be unnecessary for a
  small or simple application.
- Teams can over-abstract straightforward operations.
- Requires care to keep domain models useful rather than mirroring database
  tables or framework types.

**C# example:** `Domain` contains business rules; `Application` defines
interfaces such as `IOrderRepository`; `Infrastructure` implements them with
EF Core; `Web` handles HTTP. References point inward toward the core, while
the composition root (often `Program.cs`) wires implementations to
interfaces.

### Microservices

An application is split into small services that can be deployed
independently. Services communicate over a network, typically through HTTP
APIs or messaging, and should own their data and business capability.

**Pros**

- Services can be deployed and scaled independently.
- Teams can work with clear ownership and release boundaries.
- A problem in one service can be isolated when resilience is designed
  properly.

**Cons**

- Network calls can fail, time out, or be slow; retries and idempotency need
  careful design.
- Data consistency and transactions across services are more difficult.
- Monitoring, deployment, security, and local development require more
  infrastructure.
- Poorly chosen boundaries can create a distributed monolith with the
  complexity of services but little independent ownership.

**C# example:** separate ASP.NET Core services for Orders and Catalog,
communicating through versioned HTTP APIs or messages. Avoid sharing a
database schema as a shortcut for service-to-service access.

### Event-driven architecture

Components communicate by publishing events (facts that something happened)
and subscribing to events they need. It can be used inside one application or
between independently deployed services.

**Pros**

- Producers do not need direct knowledge of every consumer.
- Asynchronous processing can smooth traffic spikes and improve responsiveness.
- New consumers can react to existing events without changing the producer,
  when event contracts are stable.

**Cons**

- Processing is often eventually consistent, so data may not update
  immediately everywhere.
- Tracing a workflow across consumers is harder than following a direct call.
- Duplicate delivery, ordering, retries, and poison messages must be handled.

**C# example:** an Orders module publishes an `OrderPlaced` event to a broker;
an email or fulfillment consumer handles it. In .NET, a broker client or
messaging library can implement transport, while application code depends on
an abstraction where appropriate.

## Quick comparison

| Style | Typical fit | Main trade-off |
| --- | --- | --- |
| Monolith | Small-to-medium application, one team, straightforward deployment | Simple operations versus growing internal coupling |
| Modular monolith | Growing application that benefits from boundaries but not independent releases | Module discipline versus single deployment unit |
| Layered | Conventional applications with clear technical responsibilities | Familiarity versus changes crossing many layers |
| Clean / Hexagonal / Onion | Business rules that should be isolated from infrastructure | Testability and independence versus extra structure |
| Microservices | Multiple teams or capabilities needing independent deployment or scaling | Independent ownership versus distributed-systems complexity |
| Event-driven | Asynchronous workflows, integrations, or loose coupling | Decoupling and resilience versus harder debugging and consistency |

## Choosing an approach

Start with the simplest style that supports the system's current needs.
For many C# applications, a **modular monolith** with a few clear layers and
dependencies directed toward business logic is a practical default. Consider
microservices when independent deployment, scaling, or team ownership provides
enough benefit to justify the operational and distributed-systems costs.

Architecture styles are tools, not rankings. Good module boundaries,
understandable dependencies, and reliable tests matter more than adopting a
fashionable label.
