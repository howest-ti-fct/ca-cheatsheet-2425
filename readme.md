# Clean Architecture Guide with TypeScript

This guide provides an overview of Clean Architecture principles and a step-by-step approach to structuring projects into distinct layers. Following this architecture ensures scalability, maintainability, and testability by enforcing separation of concerns.

Layers:
- **Domain**: Core business logic and entities.
- **Application**: Use cases and application logic.
- **Infrastructure**: External dependencies and concrete implementations.
- **Main**: Application entry point and object creation.

---

## DOMAIN

### Description
The **Domain** layer is the core of the application, containing business rules and entities. This layer is purely logical and does not depend on external libraries, databases, or frameworks.

### Goal
- **Entities**
  - Represent core business objects with attributes and behaviors.
  - Contain domain-specific business logic.
  - Maintain consistency through aggregates and value objects.
  - Can have its own repository if it is not part of an aggregate.
- **Aggregates**
  - Group related entities together to enforce consistency.
  - Only the aggregate root has a repository.
  - Provide transactional boundaries.
- **Value Objects**
  - Represent immutable domain concepts without unique identity.
- **Repository Interfaces**
  - Define abstract data access methods.
  - Ensure the domain layer remains independent of data sources.
- **Business Rules**
  - Encapsulate logic that enforces domain constraints.

#### Unit Tests
- Validate business rules to ensure logic correctness.
- Test entities to verify behaviors and constraints.
- Ensure value objects are immutable and behave correctly.


### Step-by-Step Guide
1. Define **entities** and **value objects** as plain classes, encapsulating business logic.
2. Create **repository interfaces** with method definitions for data access.
3. Write **unit tests** to validate business logic and enforce domain constraints.

---

## APPLICATION

### Description
The **Application** layer coordinates the flow of the application by implementing use cases. It acts as an orchestrator that invokes domain logic and interacts with infrastructure components or queries data without modifying the domain.

### Goal
- **Use Cases**
  - Implement application logic using constructor-based dependency injection (e.g., repositories, unit of work).
  - Expose an `execute(input): output` method to interact with the domain or retrieve data.
  - Ensure independence from external services.

- **Unit Tests**
  - Mock queries, repositories, and unit of work to test interactions. 
  - Track method calls to verify behavior.

- **Contracts**
  - Define **ports**:
    - Unit of Work for transactional consistency.
    - Query interfaces for fetching domain data.
  - Import **repository interfaces** from the **Domain** layer into `mod.ts`.

### Step-by-Step Guide
1. Implement **use cases**, injecting queries or repositories and unit of work.
2. Define input and output interfaces for use cases.
3. Write **unit tests** using mock repositories to verify interactions and expected results.

---

## INFRASTRUCTURE

### Description
The **Infrastructure** layer provides **concrete** implementations of queries, repositories, data mappers, Web API controllers, routes, and messaging components. It integrates external dependencies such as databases, APIs, and messaging services.

### Goal
- **Repositories**
  - Implement **concrete** repository based on the interfaces.
  - Implement mappers to map entities to database models and vice versa.
  - Implement **concrete** queries based on the interfaces.
- **Web API**
  - Implement controllers that handle HTTP requests.
  - Define routes for API endpoints pointing to controllers.
- **Messaging**
  - Implement messaging controllers for asynchronous processing.
  - Define an **Async API specification** for event-driven interactions.

### Step-by-Step Guide
1. Implement persistence interfaces:
    1. If needed, implement **concrete** repository based on the **repository interfaces**, ensuring proper entity mapping and data consistency.
    2. If needed, implement **concrete** queries based on the **query interfaces** for efficient data retrieval.
2. Implement Web API (if applicable):
    1. Create **Web API controllers** to handle HTTP requests and map input/output models.
    2. Define **routes** for API endpoints following RESTful principles.
3. Implement Messaging (if applicable):
    1. Implement **messaging controllers** for handling asynchronous events.
    2. Document the messaging API using an **Async API spec** for consistency across services.

---

## MAIN

### Description
The **Main** layer is responsible for object creation and application initialization by setting up dependency injection, configuring modules, and starting the application.

### Goal
- **Persistence Module**
  - Register repository implementations in `addRepositories`.
  - Register query implementations in `addQueries`.

- **Web API Module / Messaging Module***
  - Implement `OakControllerFactory`:
    - Use a switch statement to map the `controller` to `createControllerMethod`.
    - Implement `createControllerMethod`:
        - Retrieve the query or repository from the container.
        - Retrieve the unit of work from the container if needed.
        - Create a new UseCase instance.
        - Instantiate a new controller with the UseCase.
        - Return the controller instance.


      example:
        ```TypeScript
        private async createStartTournamentController(): Promise<StartTournamentController> {
          // Retrieve the query or repository from the container.
          const tournamentRepository = (await this._serviceProvider.getService<TournamentRepository>(
              'mongoDbTournamentRepository',
          ))
              .getOrThrow();

          // Retrieve the unit of work from the container if needed.
          const unitOfWork = (await this._serviceProvider.getService<UnitOfWork>('mongoDbUnitOfWork'))
              .getOrThrow();

          // Create a new UseCase instance.
          const startTournament: StartTournament = new StartTournament(
              tournamentRepository,
              unitOfWork,
          );

          // Instantiate a new controller with the UseCase and return.
          return new StartTournamentController(
              startTournament,
          );
      }
        ```


### Step-by-Step Guide
1. Set up **dependency injection** for repositories and queries.
2. Configure **controllers** using `OakControllerFactory` to map routes to handlers.
3. Ensure **all dependencies** are correctly wired before application startup.
4. Bootstrap the application by initializing modules and starting the web server.

---

## FULL FLOW: Write Use Case in Web API

### DOMAIN
- Implement domain logic if not already present.
- Write unit tests to validate domain logic.

### APPLICATION: USE CASES (AND CONTRACTS)
- Add the repository interface if missing in the domain.
- Include the repository interface in `mod.ts` in the application layer.
- Implement a use case with **Unit of Work** and repository interaction.
- Write unit tests to verify the use case logic.

### INFRASTRUCTURE: IMPLEMENTING CONTRACTS AND CONTROLLERS
- Implement the repository in **persistence infrastructure**.
- Add a mapper for entity-to-database and database-to-entity transformations.
- Register the repository creation in the **PersistenceModule** in the **Main** layer.

### MAIN: OBJECT CREATION
- Implement a controller in **Web API infrastructure**.
- Define the route in **Web API infrastructure**.
- Register controller creation in `OakControllerFactory` within the **Main** layer.

---

## FULL FLOW: Read Use Case in Web API

### APPLICATION: USE CASES (AND CONTRACTS)
- Define the query interface in the **ports of contracts** in the application.
- Include the query interface in `mod.ts` of the application layer.
- Implement a use case for executing the query.
- Write unit tests to verify the query logic.

### INFRASTRUCTURE: IMPLEMENTING CONTRACTS AND CONTROLLERS
- Implement the query in **persistence infrastructure**.
- Register the query creation in the **PersistenceModule** in the **Main** layer.

### MAIN: OBJECT CREATION
- Implement a controller in **Web API infrastructure**.
- Define the route in **Web API infrastructure**.
- Register controller creation in `OakControllerFactory` in the **Main** layer.

---

## Summary
- **Domain**: Core business logic, entities, and business rules.
- **Application**: Use case implementations and contracts that execute business logic.
- **Infrastructure**: **concrete** implementations for queries, repositories, APIs, and messaging.
- **Main**: Application entry point for object creation with dependency injection and module initialization.
