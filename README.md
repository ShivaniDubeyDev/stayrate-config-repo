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

## Infrastructure Prerequisites

Ensure your local environments are running on their default ports before starting the ecosystem:
* **MySQL:** Port `3306` (Database: `user_db`)
* **PostgreSQL:** Port `5432` (Database: `hotel_db`, User: `postgres`)
* **MongoDB:** Port `27017` (Database: `rating_db`)
* **Java Version:** JDK 17 or JDK 23

---

## Installation & Build

Clone the repository and compile the source artifacts using the Maven wrapper. 

*(Note: If using standard Windows CMD, omit the `./` prefix and run `mvnw clean package -DskipTests`)*

```bash
# Navigate to the platform root directory
cd stayrate-platform

# Compile and package all sub-modules bypassing unit tests (PowerShell / Bash)
./mvnw clean package -DskipTests
```

## Ecosystem Startup Order

Open a separate terminal window for each service and launch them in this precise chronological order to allow service registry configurations to discover and bind properly without timing out:

```powershell
# STEP 1: Start Service Discovery Hub First
cd ServiceRegistry/ServiceRegistry
java -jar target/service-registry-0.0.1-SNAPSHOT.jar

# STEP 2: Start Centralized Configuration (Wait 10 seconds for service registry readiness)
cd ../../ConfigServer/ConfigServer
java -jar target/config-server-0.0.1-SNAPSHOT.jar

# STEP 3: Start Core Domain Instances (Can run simultaneously once Config Server is online)
cd ../../HotelService/HotelService
java -jar target/hotel-service-0.0.1-SNAPSHOT.jar

cd ../../RatingService/RatingService
java -jar target/rating-service-0.0.1-SNAPSHOT.jar

cd ../../UserService/UserService
java -jar target/user-service-0.0.1-SNAPSHOT.jar

# STEP 4: Start Routing Gateway Last (Requires downstream microservices to be available)
cd ../../ApiGateway/ApiGateway
java -jar target/api-gateway-0.0.1-SNAPSHOT.jar
```

## Verification & Monitoring

* **Eureka Dashboard:** Open http://localhost:8761 in your browser to verify that all 5 client engines show a status value of `UP`.
* **Database Inspection:**
  * Use **MySQL Workbench** to look inside `user_db`.
  * Use **pgAdmin 4** to check tables inside `hotel_db`.
  * Use **MongoDB Compass** to view runtime document collections in `rating_db`.

## Modifying Configurations

1. Apply changes to any targeted `.yml` file directly inside this repository.
2. Commit the updates to the main tracking branch (`main` or `master`).
3. Broadcast a `POST` request to the client service's Actuator endpoint (`http://localhost:<port>/actuator/refresh`) to instantly propagate updates without system reboots.
