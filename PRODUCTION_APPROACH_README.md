# How I structure this work (design note)

This is how I describe my approach when I write it up: **repositories**, **tests**, **resilience patterns**, and **libraries** I would use in a Spring Boot service.

---

## Source structure (main code)

I keep production code under one base package and split by layer:

```
src/main/java/com/example/orders/
├── OrdersApplication.java
├── controller/
├── service/
├── repository/
├── domain/
└── utility/
└── dto/
```

---

## Test structure

I mirror that layout under `src/test/java` so tests sit next to the layer they exercise:

```
src/test/java/com/example/orders/
├── controller/
└── service/
```

- I name classes **`SomethingTest`** (or `SomethingTests`) 
- **Unit tests**: pure logic, no Spring (or minimal mocks).
- **Integration tests**: SIT or SVT with external systems

---

## How I handle repositories

- I keep the **repository layer thin**: Spring Data **`JpaRepository`** (or `CrudRepository`) interfaces live under `repository/` and expose **queries and persistence only**.
- I put **business rules and transactions** in **services**, not in repositories. Repositories return entities or projections; they do not orchestrate workflows.
- For simple lookups I use **derived query methods** (`findByEmail`, `countByStatus`). For aggregates or joins I use **`@Query`** (JPQL or native) so the database does the heavy work.
- I avoid letting controllers call repositories directly; **controller → service → repository** keeps boundaries clear and tests simpler.

---

## Resilience: load balancing, caching, retry, fallback

- **Load balancing**: I run **multiple instances** behind a **gateway or load balancer** and keep the app **stateless** so traffic can spread evenly.
- **Caching**: I cache **read-heavy** data with a **TTL**; I use **Spring Cache**  (single JVM) or **Redis** (shared across instances).
- **Retry**: I retry failures, with **max attempts** and **backoff**.
- **Fallback**: After retries fail, I return a **safe degraded response** or clear error; I use a **circuit breaker** so a failing dependency is not called endlessly.

---

## Libraries I can use (Java / Spring)

| Purpose                | Libraries                                                                    |
|------------------------|------------------------------------------------------------------------------|
| **Data / repositories** | `spring-boot-starter-data-jpa`, database driver (PostgreSQL, etc.)           |
| **Testing**            | `spring-boot-starter-test` (JUnit 5, Mockito, AssertJ, MockMvc)              |
| **Resilience**         | **Resilience4j** (`retry`, `circuitbreaker`, `ratelimiter`)                  |
| **HTTP clients**       | **Spring WebClient** or **FeignClient**                                      |
| **Load balancing **    | **Cloud Loadbalancer**                                                       |
| **Caching**            | **Spring Cache**  or **Redis**  |
| **API gateway**        | **Spring Cloud Gateway**                 |
| **Observability**      | **NewRelic**, **Splunk**, Prometheus                      |

