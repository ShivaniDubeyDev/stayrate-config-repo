# StayRate Platform Centralized Configuration Repository

This repository serves as the single source of truth for the externalized configuration profiles of the StayRate microservices ecosystem. These properties are dynamically fetched by the **Config Server** at runtime.

## Core Infrastructure Layout

| Service Module | Execution Port | Registration Protocol | Configuration Strategy |
| :--- | :--- | :--- | :--- |
| **`service-registry`** | `8761` | Standalone Discovery Hub | Local Core Bootstrap Configuration |
| **`config-server`** | `8085` | Discovery Client Instance | Local Git Bootstrap Target Pointer Configuration |
| **`api-gateway`** | `8087` | Discovery Client Instance | Managed Centralized Configuration (`api-gateway.yml`) |
| **`user-service`** | `8081` | Discovery Client Instance | Managed Centralized Configuration (`user-service.yml`) |
| **`hotel-service`** | `8082` | Discovery Client Instance | Managed Centralized Configuration (`hotel-service.yml`) |
| **`rating-service`** | `8083` | Discovery Client Instance | Managed Centralized Configuration (`rating-service.yml`) |

---

## Managed Profiles & Properties

### 1. `api-gateway.yml`
* **Routing Table:** Non-blocking reactive routes targeting service identities (`lb://USER-SERVICE`, `lb://HOTEL-SERVICE`, `lb://RATING-SERVICE`).
* **Security:** Reactive WebFlux OAuth2 Gateway handling single-sign-on (SSO) routines with Okta.

### 2. `user-service.yml`
* **Storage Engine:** Relational MySQL instance configuration targeting `user_db`.
* **Security Context:** Spring Security Resource Server configurations with specialized downstream `client_credentials` scopes for inter-service communication.
* **Resilience Profile:** Complete Resilience4j suite including a Count-Based Circuit Breaker (`ratingHotelBreaker`), Retry backoffs, and Rate Limiter (`userRateLimiter`).

### 3. `hotel-service.yml`
* **Storage Engine:** Relational PostgreSQL instance configuration targeting `hotel_db`.
* **Security Context:** Jakarta namespace-aligned JWT access token validation configurations via Okta.

### 4. `rating-service.yml`
* **Storage Engine:** NoSQL MongoDB instance connection string targeting `rating_db`.
* **Security Context:** Okta OAuth2 token enforcement configuration layer.

---

## Execution & Operational Order

To launch the microservices architecture correctly without dependency failures, boot the components in this exact order:

1. **Start Infrastructure Components:** Verify local instances of MySQL, PostgreSQL, and MongoDB are active.
2. **Start Service Registry:** Run `ServiceRegistryApplication` (`8761`). Verify the dashboard via browser.
3. **Start Config Server:** Run `ConfigServerApplication` (`8085`). Ensure it successfully clones this GitHub repository.
4. **Start API Gateway:** Run `ApiGatewayApplication` (`8087`).
5. **Start Domain Core Services:** Run `UserServiceApplication` (`8081`), `HotelServiceApplication` (`8082`), and `RatingServiceApplication` (`8083`) in any sequence.

---

## Modifying Configurations

1. Apply changes to any targeted `.yml` file directly inside this repository.
2. Commit the updates to the main tracking branch (`main` or `master`).
3. Broadcast a `POST` request to the client service's Actuator endpoint (`http://localhost:<port>/actuator/refresh`) to instantly propagate updates without system reboots.
