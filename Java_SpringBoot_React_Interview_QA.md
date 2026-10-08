# Java + Spring Boot + React Full-Stack Interview Questions & Answers

## Role Focus

**Experience:** 4–7 years  
**Primary stack:** Java 8+, Spring Boot, Microservices, REST, JPA/Hibernate, SQL/NoSQL, React, TypeScript, JavaScript ES6+, Redux/Context, OAuth2/OIDC/JWT, Jest/RTL, Cypress/Playwright, Docker/Cloud  
**Interview emphasis:** clean code, design decisions, troubleshooting, security, performance, ownership, and communication.

---

# 1. Core Java

## Q1. What are the main OOP principles in Java?

**Answer:**
The four main principles are:

- **Encapsulation:** Keep data and behavior together and restrict direct access using access modifiers.
- **Abstraction:** Expose only required behavior through interfaces or abstract classes.
- **Inheritance:** Reuse or extend behavior from a parent class.
- **Polymorphism:** The same interface can have different implementations, mainly through method overriding.

In production code, I prefer composition over inheritance when it reduces coupling and makes the design easier to maintain.

---

## Q2. What is the difference between `==` and `.equals()`?

**Answer:**

- `==` compares primitive values or object references.
- `.equals()` compares logical equality when the class overrides it.

For example, two different `String` objects can contain the same value. `.equals()` checks their content, while `==` checks whether both references point to the same object.

---

## Q3. Why must `equals()` and `hashCode()` be overridden together?

**Answer:**
Hash-based collections such as `HashMap` and `HashSet` depend on both methods.

If two objects are equal according to `equals()`, they must return the same `hashCode()`. Otherwise, collections may fail to find an object even though it is logically equal.

---

## Q4. Explain `final`, `finally`, and `finalize()`.

**Answer:**

- `final` prevents reassignment, overriding, or inheritance depending on where it is used.
- `finally` is a block normally executed after `try/catch`, commonly used for cleanup.
- `finalize()` was a legacy garbage-collection hook and should not be used. It has been deprecated/removed from modern Java usage patterns.

---

## Q5. Checked vs unchecked exceptions?

**Answer:**

**Checked exceptions** are verified by the compiler and must be handled or declared. Examples include `IOException`.

**Unchecked exceptions** extend `RuntimeException` and generally represent programming errors or invalid state, such as `NullPointerException` or `IllegalArgumentException`.

In REST applications, I normally use meaningful domain/runtime exceptions and map them centrally using `@ControllerAdvice`.

---

## Q6. What is the Java Collections Framework?

**Answer:**
It provides standard data structures and algorithms.

Common interfaces:

- `List` — ordered collection, duplicates allowed.
- `Set` — unique elements.
- `Map` — key-value pairs.
- `Queue`/`Deque` — processing elements in queue/deque order.

Common implementations include `ArrayList`, `LinkedList`, `HashSet`, `TreeSet`, `HashMap`, `TreeMap`, and `ConcurrentHashMap`.

---

## Q7. ArrayList vs LinkedList?

**Answer:**

`ArrayList` provides fast random access, generally O(1), and is usually my default choice.

`LinkedList` can insert/remove efficiently when the node position is already known, but finding a position is O(n) and it has higher memory overhead.

For most application code, `ArrayList` is preferable unless there is a specific workload that benefits from a linked structure.

---

## Q8. How does HashMap work internally?

**Answer:**
A `HashMap` uses the key's hash code to determine a bucket. It then compares keys using `equals()` when resolving collisions.

Modern Java implementations can use tree-based structures for heavily collided buckets, improving worst-case lookup behavior.

Good `hashCode()` and `equals()` implementations are therefore important.

---

## Q9. HashMap vs ConcurrentHashMap?

**Answer:**

`HashMap` is not thread-safe.

`ConcurrentHashMap` supports concurrent access with synchronization mechanisms designed for high concurrency.

For shared mutable state accessed by multiple threads, I would consider `ConcurrentHashMap`, but I would first ask whether shared mutable state is actually required.

---

## Q10. What is immutability?

**Answer:**
An immutable object cannot change after construction.

A common example is `String`.

Benefits include:

- Thread safety
- Easier reasoning
- Safe sharing
- Better suitability as map keys
- Reduced accidental state changes

For a custom immutable class, I generally make the class final, fields private/final, initialize through the constructor, and avoid exposing mutable internal objects directly.

---

# 2. Java 8: Streams, Lambdas and Optional

## Q11. What is a functional interface?

**Answer:**
A functional interface contains exactly one abstract method.

Examples:

- `Predicate<T>`
- `Function<T,R>`
- `Consumer<T>`
- `Supplier<T>`

They are commonly used with lambda expressions.

---

## Q12. What are Java Streams?

**Answer:**
Streams provide a declarative way to process collections and other data sources.

Typical operations include:

```java
orders.stream()
      .filter(o -> o.getAmount() > 1000)
      .map(Order::getAmount)
      .reduce(BigDecimal.ZERO, BigDecimal::add);
```

Intermediate operations such as `filter` and `map` are lazy. Terminal operations such as `collect`, `reduce`, and `forEach` trigger execution.

---

## Q13. `map()` vs `flatMap()`?

**Answer:**

`map()` transforms each element into one element.

`flatMap()` transforms each element into a stream and then flattens the resulting streams.

Example:

```java
List<List<String>> names = ...;

List<String> result = names.stream()
        .flatMap(List::stream)
        .toList();
```

---

## Q14. What is Optional and when should you use it?

**Answer:**
`Optional<T>` represents a value that may or may not exist.

It helps make absence explicit and can reduce accidental null handling.

I mainly use it for return values where absence is a valid business outcome. I avoid using `Optional` indiscriminately for entity fields, method parameters, or serialization models.

---

## Q15. How would you group orders by customer and calculate total price?

**Answer:**

```java
Map<Long, BigDecimal> totals =
    orders.stream()
          .collect(Collectors.groupingBy(
              Order::getCustomerId,
              Collectors.reducing(
                  BigDecimal.ZERO,
                  Order::getPrice,
                  BigDecimal::add
              )
          ));
```

This is readable and performs the aggregation in one stream pipeline.

---

# 3. Concurrency and Multithreading

## Q16. Process vs thread?

**Answer:**
A process has its own memory space. Threads run within a process and share its memory.

Threads are cheaper than processes but shared state introduces synchronization and concurrency concerns.

---

## Q17. What is a race condition?

**Answer:**
A race condition occurs when multiple threads access shared mutable state and the result depends on execution timing.

I prevent it through immutable data, thread-safe collections, synchronization, locks, atomic classes, or redesigning the operation to avoid shared mutable state.

---

## Q18. `synchronized` vs `volatile`?

**Answer:**

`volatile` provides visibility of updates between threads but does not make compound operations atomic.

`synchronized` provides mutual exclusion and visibility guarantees.

For example, `count++` is not made atomic simply by declaring `count` volatile. For counters, `AtomicInteger` may be more appropriate.

---

## Q19. What is ExecutorService?

**Answer:**
`ExecutorService` manages a pool of worker threads and allows tasks to be submitted without manually creating and managing every thread.

It improves resource management and makes asynchronous execution easier to control.

---

## Q20. What are common concurrency problems?

**Answer:**

- Race conditions
- Deadlocks
- Starvation
- Thread contention
- Excessive context switching
- Incorrect shared-state management

In production troubleshooting, I look at thread dumps, CPU usage, blocked/waiting threads, lock contention, and application metrics.

---

# 4. Clean Code and Refactoring

## Q21. How do you identify design flaws in an existing codebase?

**Answer:**
I first understand the business flow and identify high-change/high-risk areas.

Then I look for:

- Large classes/methods
- Duplicate logic
- High coupling
- Low cohesion
- Deep nesting
- Hidden side effects
- Poor exception handling
- Repeated database calls
- God classes
- Difficult-to-test code

I refactor incrementally, keeping tests around the existing behavior so that changes remain safe.

---

## Q22. Explain SOLID principles.

**Answer:**

- **S — Single Responsibility:** A class should have one primary reason to change.
- **O — Open/Closed:** Open for extension, closed for unnecessary modification.
- **L — Liskov Substitution:** Subtypes should be safely substitutable for their base types.
- **I — Interface Segregation:** Prefer focused interfaces over large interfaces.
- **D — Dependency Inversion:** Depend on abstractions rather than concrete implementations.

---

## Q23. How do you choose between multiple design approaches?

**Answer:**
I compare them using:

1. Functional requirements
2. Performance
3. Security
4. Complexity
5. Maintainability
6. Scalability
7. Operational impact
8. Team familiarity
9. Testing effort
10. Cost

I document important trade-offs rather than choosing an approach only because it is familiar.

---

# 5. Spring Framework and Spring Boot

## Q24. What is dependency injection?

**Answer:**
Dependency Injection means an object receives its dependencies from outside instead of constructing them itself.

Constructor injection is generally preferred because dependencies are explicit and required dependencies can be immutable.

Example:

```java
@Service
public class OrderService {
    private final PaymentClient paymentClient;

    public OrderService(PaymentClient paymentClient) {
        this.paymentClient = paymentClient;
    }
}
```

---

## Q25. What is a Spring Bean?

**Answer:**
A Spring Bean is an object managed by the Spring IoC container.

Beans can be created using annotations such as `@Component`, `@Service`, `@Repository`, `@Controller`, or through `@Bean` methods.

---

## Q26. Explain common Spring annotations.

**Answer:**

- `@Component` — generic Spring-managed component.
- `@Service` — service/business layer.
- `@Repository` — persistence layer.
- `@RestController` — REST controller.
- `@Configuration` — configuration class.
- `@Bean` — explicitly declares a bean.
- `@Autowired` — dependency injection, though constructor injection is preferred.
- `@Transactional` — transaction boundary.

---

## Q27. Explain Spring Bean lifecycle.

**Answer:**
At a high level:

1. Spring creates the bean.
2. Dependencies are injected.
3. Initialization callbacks/processors run.
4. Bean becomes available for use.
5. Destruction callbacks run when the context shuts down.

Lifecycle hooks such as `@PostConstruct` and `@PreDestroy` can be used when appropriate.

---

## Q28. What is Spring Boot?

**Answer:**
Spring Boot simplifies Spring application development by providing auto-configuration, starter dependencies, embedded servers, externalized configuration, production features and conventions.

It reduces boilerplate while still allowing explicit configuration when required.

---

## Q29. What does `@Transactional` do?

**Answer:**
It defines a transactional boundary. If an operation succeeds, changes are committed; when configured exceptions cause rollback, changes are rolled back.

Example:

```java
@Transactional
public void transferMoney(...) {
    debit();
    credit();
}
```

The exact rollback behavior depends on exception type and transaction configuration.

---

## Q30. What is a common `@Transactional` pitfall?

**Answer:**
Self-invocation can bypass Spring's proxy-based transaction interception.

Other issues include:

- Long-running transactions
- Incorrect transaction boundaries
- Lazy loading outside a transaction
- Unexpected rollback behavior
- Calling external services while holding database transactions open

---

# 6. REST API Design

## Q31. What makes a REST API well designed?

**Answer:**

- Resource-oriented URLs
- Correct HTTP methods
- Appropriate status codes
- Consistent error responses
- Validation
- Pagination/filtering/sorting
- Authentication and authorization
- Idempotency where appropriate
- Versioning strategy
- Clear API documentation
- Observability

---

## Q32. PUT vs PATCH?

**Answer:**

`PUT` generally represents replacement of a resource and is expected to be idempotent.

`PATCH` represents a partial update.

The exact semantics should be documented consistently by the API.

---

## Q33. How do you handle REST errors?

**Answer:**
I use centralized exception handling with `@RestControllerAdvice`.

I return a consistent structure containing information such as:

- Timestamp
- HTTP status
- Error code
- User-safe message
- Correlation/request ID

Internal stack traces and sensitive details should not be exposed to clients.

---

## Q34. How do you design pagination?

**Answer:**
For simple APIs I can use page/size parameters. For very large or frequently changing datasets, cursor/keyset pagination can be more efficient and stable than large offset values.

I also define maximum page sizes to prevent expensive queries.

---

# 7. Microservices

## Q35. What is microservices architecture?

**Answer:**
Microservices decomposes an application into independently deployable services aligned around business capabilities.

Benefits include independent deployment and scaling.

Challenges include distributed transactions, network failures, observability, deployment complexity and data consistency.

---

## Q36. How do you define service boundaries?

**Answer:**
I start from business capabilities and domain boundaries rather than technical layers.

I consider:

- Business ownership
- Data ownership
- Change frequency
- Transaction boundaries
- Team ownership
- Communication patterns
- Scalability requirements

A service should ideally own its data and expose business capabilities through APIs/events.

---

## Q37. How do microservices communicate?

**Answer:**
Common approaches are:

- Synchronous REST/HTTP
- gRPC
- Asynchronous messaging/events through brokers

I choose based on latency, reliability, coupling, ordering, throughput and consistency requirements.

---

## Q38. How do you handle data consistency between services?

**Answer:**
I avoid distributed database transactions where possible.

Depending on the use case, I use:

- Events
- Outbox pattern
- Saga pattern
- Idempotent consumers
- Retry mechanisms
- Compensating transactions

The business should explicitly define whether consistency must be immediate or eventual.

---

## Q39. What is the Circuit Breaker pattern?

**Answer:**
A circuit breaker prevents repeated calls to an unhealthy downstream service.

Typical states are:

- Closed
- Open
- Half-open

It helps prevent cascading failures and allows a system to fail fast or use a fallback when appropriate.

---

## Q40. How do you make microservice calls resilient?

**Answer:**
I consider:

- Timeouts
- Retries with backoff
- Circuit breakers
- Bulkheads
- Rate limits
- Idempotency
- Fallbacks where meaningful
- Monitoring and tracing

Retries should not blindly be applied to every failure because they can amplify an outage.

---

# 8. Spring Security, OAuth2, OIDC and JWT

## Q41. Authentication vs authorization?

**Answer:**

**Authentication:** Who are you?

**Authorization:** What are you allowed to do?

For example, OAuth2/OIDC can establish identity, while application roles/permissions determine whether the user can access an administrative API.

---

## Q42. What is JWT?

**Answer:**
JWT is a signed token containing claims.

A typical JWT has:

- Header
- Payload
- Signature

The server can validate the signature and claims such as issuer, audience and expiration.

JWTs are signed, not automatically encrypted, so sensitive information should not be placed in the payload unless appropriate protection exists.

---

## Q43. OAuth2 vs OIDC?

**Answer:**
OAuth2 is primarily an authorization framework.

OpenID Connect (OIDC) adds an identity layer on top of OAuth2 and provides standardized authentication information, including the ID token.

---

## Q44. How should tokens be handled securely in a React application?

**Answer:**
The approach depends on the architecture.

For browser applications, I prefer secure server-side/session or carefully designed cookie-based approaches where possible. If tokens are stored in the browser, I minimize exposure, use short-lived access tokens, protect refresh tokens appropriately, and never put tokens in URLs.

I also use HTTPS, CSP, secure cookie attributes where applicable, CSRF protection where applicable, and strict XSS defenses.

---

# 9. SQL, Databases and JPA

## Q45. What is normalization?

**Answer:**
Normalization organizes relational data to reduce unnecessary duplication and update anomalies.

Common normal forms include 1NF, 2NF and 3NF.

In production systems, I balance normalization with read performance and reporting requirements rather than blindly normalizing everything.

---

## Q46. How do you troubleshoot a slow SQL query?

**Answer:**

1. Reproduce the query and measure execution time.
2. Inspect the execution plan.
3. Check indexes.
4. Check joins and filtering.
5. Look for unnecessary columns/rows.
6. Check statistics and cardinality.
7. Investigate locking/contention.
8. Consider query/data-model changes.
9. Re-test using realistic data volume.

---

## Q47. What is an index?

**Answer:**
An index is a data structure that helps the database locate rows efficiently.

Indexes improve reads but add storage and write/update overhead.

I create indexes based on actual query patterns and validate them using execution plans.

---

## Q48. What is the N+1 query problem in JPA?

**Answer:**
It happens when an initial query loads N entities and then additional queries are executed for each entity to load related data.

Possible solutions include:

- Fetch joins
- Entity graphs
- DTO projections
- Batch fetching
- Carefully selected fetch strategies

The solution should be based on the actual query use case rather than making every relationship eager.

---

## Q49. Lazy vs eager loading?

**Answer:**

**Lazy:** Related data is loaded when accessed.

**Eager:** Related data is loaded immediately.

Lazy loading is often safer for large relationships, but it must be designed carefully to avoid N+1 queries or lazy initialization problems.

---

## Q50. Optimistic vs pessimistic locking?

**Answer:**

**Optimistic locking** assumes conflicts are uncommon and typically uses a version field such as `@Version`.

**Pessimistic locking** locks database rows to prevent conflicting changes.

Optimistic locking is often preferable for scalable applications when conflicts are relatively rare.

---

# 10. Testing and Debugging

## Q51. Unit test vs integration test?

**Answer:**

A unit test tests a small unit in isolation, usually with mocks for external dependencies.

An integration test verifies interaction between components such as a service, database, HTTP layer or messaging system.

Both are important. Unit tests give fast feedback, while integration tests validate real integration behavior.

---

## Q52. JUnit and Mockito?

**Answer:**
JUnit is the testing framework.

Mockito is commonly used to mock dependencies.

Example:

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private PaymentClient paymentClient;

    @InjectMocks
    private OrderService orderService;
}
```

I focus tests on business behavior rather than implementation details.

---

## Q53. What makes a good unit test?

**Answer:**

- Clear arrange/act/assert structure
- One behavior per test
- Meaningful test names
- Deterministic execution
- Minimal mocking
- No unnecessary implementation coupling
- Coverage of success and failure paths

---

## Q54. How do you troubleshoot a production issue?

**Answer:**

1. Understand the impact and scope.
2. Check logs, metrics and traces.
3. Identify when the issue started.
4. Compare recent deployments/configuration changes.
5. Reproduce where possible.
6. Isolate the failing component.
7. Apply the safest mitigation.
8. Validate recovery.
9. Perform root-cause analysis.
10. Add tests/monitoring/process improvements to prevent recurrence.

I avoid making unverified changes directly in production.

---

# 11. React and TypeScript

## Q55. What is React?

**Answer:**
React is a component-based UI library for building interactive user interfaces.

Applications are composed from reusable components, with state and props controlling rendering.

---

## Q56. Props vs state?

**Answer:**

**Props** are inputs passed from a parent component.

**State** is data managed by a component or state-management system.

A child should generally not mutate its props directly.

---

## Q57. Explain React hooks.

**Answer:**
Common hooks include:

- `useState` — local state.
- `useEffect` — synchronize with external systems.
- `useMemo` — memoize expensive computed values.
- `useCallback` — memoize function references.
- `useRef` — persist mutable values or reference DOM elements.
- Custom hooks — reuse stateful logic.

Hooks should follow the Rules of Hooks and be used consistently.

---

## Q58. What is `useEffect` commonly used for?

**Answer:**
`useEffect` is used to synchronize a component with an external system, such as subscriptions, browser APIs or network interactions.

I avoid using effects for calculations that can be derived directly during rendering.

---

## Q59. What causes unnecessary React re-renders?

**Answer:**
Common causes include:

- State updates
- Parent re-renders
- Changing object/function references
- Poor component boundaries
- Broad context updates
- Large state scopes

I first profile the application before adding memoization.

---

## Q60. `useMemo` vs `useCallback`?

**Answer:**

`useMemo` memoizes a computed value.

`useCallback` memoizes a function reference.

Neither should be added everywhere. I use them when profiling or component behavior shows that avoiding recalculation/reference changes provides a real benefit.

---

# 12. React State Management

## Q61. Context API vs Redux Toolkit/Zustand?

**Answer:**
Context is useful for relatively stable cross-cutting values such as theme, locale or authentication context.

For complex shared application state, Redux Toolkit or Zustand can provide better structure, debugging and state update patterns.

I choose based on state complexity rather than introducing a state library automatically.

---

## Q62. How do you handle API data fetching?

**Answer:**
I separate server state from UI state.

For server state, a data-fetching library can provide caching, retries, invalidation and loading/error states.

For simpler applications, a dedicated service layer with hooks can be enough.

I also handle cancellation/race conditions, pagination, errors and stale data.

---

# 13. TypeScript

## Q63. Why use TypeScript with React?

**Answer:**
TypeScript adds static typing, which improves IDE support, refactoring safety and detection of incorrect data usage before runtime.

It is particularly valuable for large React applications and API contracts.

---

## Q64. `interface` vs `type`?

**Answer:**
Both can describe object shapes.

`interface` is commonly used for extensible object contracts.

`type` is more flexible for unions, intersections, tuples and other type compositions.

The important point is consistent usage and accurate modeling rather than treating one as universally superior.

---

# 14. React API Integration

## Q65. How do you integrate a React application with a REST API?

**Answer:**
I create a typed API/service layer, define request/response models, centralize common HTTP behavior, and expose data through hooks or state management.

I handle:

- Loading states
- Empty states
- API errors
- Authentication
- Pagination
- Retries where appropriate
- Cancellation
- Validation
- Contract changes

For larger teams, OpenAPI-generated types/clients can reduce frontend/backend contract drift.

---

## Q66. How do you handle authentication in React?

**Answer:**
I separate authentication state from authorization.

The application obtains identity/session information from the authentication mechanism, then uses role/permission information to control UI behavior.

Route guards improve UX, but backend authorization must always enforce the actual security boundary.

---

# 15. React Performance

## Q67. How do you optimize a slow React application?

**Answer:**

1. Profile the application.
2. Identify expensive renders.
3. Split large components.
4. Reduce unnecessary state updates.
5. Use memoization selectively.
6. Lazy-load routes/heavy components.
7. Code split bundles.
8. Virtualize large lists.
9. Optimize API requests and caching.
10. Compress and optimize assets.

I measure before and after rather than optimizing based only on assumptions.

---

## Q68. What is code splitting?

**Answer:**
Code splitting divides JavaScript into smaller chunks that can be loaded when needed.

Route-level lazy loading is a common example.

This reduces the initial JavaScript payload and can improve startup performance.

---

## Q69. How do you render thousands of rows efficiently?

**Answer:**
I avoid rendering every row at once.

I consider:

- Pagination
- Virtualization/windowing
- Server-side filtering/sorting
- Debounced search
- Memoized row components
- Efficient keys
- Smaller API payloads

---

# 16. HTML, CSS and Accessibility

## Q70. Semantic HTML?

**Answer:**
Semantic elements communicate the meaning and structure of content.

Examples:

`header`, `nav`, `main`, `section`, `article`, `footer`, `button`, and `form`.

Semantic HTML improves accessibility, maintainability and SEO.

---

## Q71. How do you build accessible React applications?

**Answer:**
I use semantic HTML first, keyboard navigation, visible focus states, correct labels, appropriate ARIA only when needed, meaningful error messages, sufficient contrast and screen-reader-friendly structure.

I also test keyboard flows and accessibility tooling.

---

# 17. Build Tools and Deployment

## Q72. Vite vs Webpack?

**Answer:**
Vite provides a fast development experience using native ESM-based development and optimized production builds.

Webpack is a highly configurable mature bundler with a large ecosystem.

The choice depends on project requirements, existing infrastructure and team familiarity.

---

## Q73. What are environment variables?

**Answer:**
They allow environment-specific configuration without hardcoding values into source code.

Examples include API URLs and feature configuration.

Secrets should never be exposed through frontend bundles because browser-delivered environment variables are effectively public.

---

# 18. Security

## Q74. What are common API security practices?

**Answer:**

- HTTPS
- Strong authentication
- Authorization at every protected endpoint
- Input validation
- Output encoding where applicable
- Rate limiting
- Secure headers
- Proper CORS configuration
- Protection against injection
- Secure secret management
- Audit logging
- Dependency vulnerability scanning

---

## Q75. How do you prevent SQL injection?

**Answer:**
Use parameterized queries/prepared statements and ORM/query APIs correctly.

Never concatenate untrusted input into SQL strings.

Input validation is useful, but it should not replace parameterization.

---

## Q76. What is CORS?

**Answer:**
Cross-Origin Resource Sharing controls which browser origins can access resources from another origin.

I configure allowed origins, methods and headers explicitly rather than allowing every origin in production.

---

# 19. System Design and Architecture

## Q77. Design a scalable order management system.

**Answer:**
I would start with services such as:

- Order Service
- Customer Service
- Inventory Service
- Payment Service
- Notification Service

The Order Service owns order state.

For synchronous operations I could use REST, while important domain events such as `OrderCreated` can be published asynchronously.

For consistency, I would consider the Saga and Outbox patterns.

I would add:

- Authentication/authorization
- API gateway where appropriate
- Centralized configuration
- Observability
- Metrics
- Distributed tracing
- Resilience patterns
- Database indexes
- Caching where justified

The exact architecture would depend on scale, availability and business consistency requirements.

---

# 20. Performance

## Q78. How do you investigate high API latency?

**Answer:**
I break the request into components:

- API processing time
- Database time
- External service time
- Serialization
- Network time
- Thread/connection pool wait

Then I use metrics, logs, traces and profiling to identify the dominant bottleneck.

I optimize the actual bottleneck rather than prematurely changing architecture.

---

## Q79. What causes connection pool exhaustion?

**Answer:**
Possible causes include:

- Slow queries
- Long transactions
- Connection leaks
- Too many concurrent requests
- External calls performed inside transactions
- Pool configured too small
- Database capacity limitations

I check pool metrics, active/idle connections, query latency and transaction duration.

---

# 21. Agile / Scrum

## Q80. What is your experience with Agile?

**Answer:**
I work in iterative delivery cycles where requirements are broken into smaller deliverables.

I participate in:

- Sprint planning
- Daily stand-ups
- Estimation
- Backlog refinement
- Reviews/demos
- Retrospectives

I focus on delivering working increments while communicating risks and dependencies early.

---

## Q81. How do you handle unclear requirements?

**Answer:**
I ask targeted questions about the business objective, users, expected behavior, edge cases, non-functional requirements and acceptance criteria.

If assumptions are unavoidable, I document them and confirm them with the stakeholder before implementation.

---

# 22. Client and Stakeholder Questions

## Q82. How do you explain a technical issue to a non-technical stakeholder?

**Answer:**
I avoid implementation jargon and explain the business impact first.

For example:

"The service is responding slowly because the database query is processing more records than expected. I am optimizing the query and adding the required index. This should reduce response time without changing the user-facing behavior."

---

## Q83. What do you do when you disagree with a technical decision?

**Answer:**
I explain my concern using measurable factors such as performance, security, maintainability or delivery risk.

I listen to the other viewpoint and compare alternatives objectively. Once the decision is made, I support the agreed approach unless there is a serious risk that needs escalation.

---

# 23. Scenario-Based Questions

## Q84. A production API suddenly becomes slow. What do you do?

**Answer:**
First I determine the scope and impact. Then I check dashboards, logs, traces, database performance, external dependencies and recent deployments.

If a recent deployment caused the issue, I evaluate rollback or mitigation. After stabilization, I identify the root cause and add tests/monitoring to prevent recurrence.

---

## Q85. A downstream microservice is unavailable. What should your service do?

**Answer:**
It depends on business criticality.

I would use a reasonable timeout, limited retry with backoff where safe, circuit breaker, and a fallback only if the business behavior supports one.

For non-critical operations, asynchronous processing may be better than blocking the user request.

---

## Q86. Your React page takes 5 seconds to load. How do you debug it?

**Answer:**
I use browser performance tools and React profiling.

I check:

- Bundle size
- Network requests
- API latency
- Render time
- Large dependencies
- Images/assets
- JavaScript execution
- Unnecessary component rendering

Then I apply targeted improvements such as lazy loading, code splitting, caching, API optimization and component optimization.

---

## Q87. A database query works in development but is slow in production. Why?

**Answer:**
Production may have much larger data volume, different indexes/statistics, different query plans, hardware constraints, locking, concurrency or different configuration.

I compare execution plans and database metrics using production-like data rather than assuming the query itself is always the problem.

---

# 24. Rapid-Fire Questions

## Q88. What is dependency inversion?

Depend on abstractions rather than concrete implementations.

## Q89. What is idempotency?

Repeating the same request produces the same intended final state.

## Q90. What is caching?

Storing frequently accessed data closer to consumers to reduce expensive computation or I/O.

## Q91. What is a DTO?

A Data Transfer Object used to carry data across application boundaries without exposing internal domain/entity structures.

## Q92. What is an API gateway?

A centralized entry point that can handle routing, authentication, rate limiting, request transformation and other cross-cutting concerns.

## Q93. What is observability?

The ability to understand system behavior using logs, metrics and distributed traces.

## Q94. What is Docker?

A containerization technology that packages an application and its dependencies into a reproducible runtime unit.

## Q95. What is CI/CD?

Continuous Integration validates changes automatically; Continuous Delivery/Deployment automates releasing changes through environments.

---

# 25. Questions You Should Ask the Interviewer

1. What are the main backend and frontend responsibilities of this role?
2. How are microservice boundaries currently defined?
3. What are the biggest technical challenges the team is solving?
4. What is the deployment and CI/CD process?
5. Which cloud services are most important for this application?
6. How are API contracts managed between React and backend teams?
7. What testing strategy is used for unit, integration and E2E testing?
8. How does the team handle production incidents and observability?
9. What would success look like in the first 3–6 months?
10. What technical debt or modernization work is currently planned?

---

# 26. 60-Second Self Introduction Template

> "I am a software engineer with around 5 years of experience working across Java backend and React frontend development. My backend experience includes Core Java, Spring Boot, REST APIs, microservices, JPA/Hibernate and SQL. On the frontend, I have worked with React, JavaScript/TypeScript, state management and REST API integration.
>
> I focus on writing maintainable code, designing clean APIs, troubleshooting production issues and improving performance. I am also comfortable working with cross-functional teams, understanding business requirements and delivering features iteratively in Agile environments.
>
> In my recent projects, I have worked on scalable applications where frontend and backend services communicate through REST APIs, with attention to security, testing and deployment."

---

# 27. Final Interview Preparation Checklist

## Java
- [ ] OOP and SOLID
- [ ] Collections
- [ ] HashMap internals
- [ ] Exception handling
- [ ] Java 8 Streams
- [ ] Lambdas
- [ ] Optional
- [ ] Multithreading
- [ ] ConcurrentHashMap
- [ ] ExecutorService
- [ ] Immutability

## Spring Boot
- [ ] Dependency Injection
- [ ] Bean lifecycle
- [ ] Core annotations
- [ ] REST APIs
- [ ] Validation
- [ ] Exception handling
- [ ] Transactions
- [ ] Spring Security
- [ ] Spring Cloud
- [ ] Resilience patterns

## Microservices
- [ ] Service boundaries
- [ ] REST communication
- [ ] Messaging
- [ ] Data consistency
- [ ] Saga
- [ ] Outbox
- [ ] Circuit breaker
- [ ] Retry/timeout
- [ ] Observability

## Database
- [ ] SQL joins
- [ ] Indexes
- [ ] Query optimization
- [ ] Execution plans
- [ ] Normalization
- [ ] Transactions
- [ ] JPA/Hibernate
- [ ] N+1 problem
- [ ] Optimistic locking

## React
- [ ] JSX
- [ ] Props/state
- [ ] Hooks
- [ ] Rendering behavior
- [ ] Context
- [ ] Redux Toolkit/Zustand
- [ ] API integration
- [ ] TypeScript
- [ ] Authentication
- [ ] Performance
- [ ] Accessibility

## Testing
- [ ] JUnit
- [ ] Mockito
- [ ] Integration tests
- [ ] Jest
- [ ] React Testing Library
- [ ] Cypress/Playwright

## Behavioral
- [ ] Production incident example
- [ ] Difficult bug example
- [ ] Refactoring example
- [ ] Performance improvement example
- [ ] Stakeholder conflict example
- [ ] Ambiguous requirement example
- [ ] Ownership example
- [ ] Team collaboration example

---

# 28. Best Strategy for This Interview

For a 4–7 year role, do not answer only with definitions.

Use this structure:

**Concept → Real project example → Trade-off → Result**

Example:

> "We had a slow API caused by an inefficient database query. I analyzed the execution plan, found that the filtering column was not indexed correctly, optimized the query and added the appropriate index. I also verified that the index did not negatively affect write performance. After testing with production-like data, the API response time improved significantly."

Prepare at least **5 real project stories** covering:

1. A difficult production issue
2. A performance optimization
3. A refactoring/clean-code improvement
4. A microservice/API design decision
5. A React feature or frontend performance improvement

These stories can be reused across many behavioral and technical questions.
