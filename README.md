# Salon Booking System

A full-stack salon booking application with a React frontend and Spring Boot microservices backend. Customers can discover salons, browse services, make bookings, pay for appointments, receive notifications, and submit reviews. Salon owners and administrators have separate management features.

## Architecture

- **Frontend:** React 18 with Create React App, Material UI, Redux, and Axios
- **Backend:** Spring Boot microservices with Spring Cloud Gateway, Eureka service discovery, Spring Data JPA, RabbitMQ, and Keycloak integration
- **Databases:** MySQL databases used by the backend services
- **Payments:** Razorpay and Stripe integration points
- **Deployment support:** Docker Compose configuration for infrastructure and prebuilt service images

Backend services:

| Service | Source directory | Port |
| --- | --- | ---: |
| API Gateway | `backend (microservices)/gateway-server` | 5000 |
| User service | `backend (microservices)/user-service` | 5001 |
| Salon service | `backend (microservices)/salon` | 5002 |
| Booking service | `backend (microservices)/booking` | 5003 |
| Category service | `backend (microservices)/category` | 5004 |
| Service-offering service | `backend (microservices)/service-offering` | 5005 |
| Payment service | `backend (microservices)/payment` | 5006 |
| Notification service | `backend (microservices)/notifications` | 5007 |
| Review service | `backend (microservices)/review` | 5008 |
| Eureka server | `backend (microservices)/eurekaserver` | 8070 |

The frontend uses the API gateway at `http://localhost:5000`.

## Technology Requirements

Install the following before starting the project:

| Technology | Minimum version | Version used or configured by this repository |
| --- | --- | --- |
| Git | 2.30+ | Any recent version should work |
| Java JDK | 17 | Java 17, configured in every backend `pom.xml` |
| Maven | 3.9.9 | Maven Wrapper downloads Maven 3.9.9 |
| Node.js | 14.0.0+ | Node.js 18 or 20 LTS is recommended |
| npm | 6+ | Installed with Node.js |
| Docker Engine | 20.10+ | Required for the Compose workflow |
| Docker Compose | Compose v2 | Required for the Compose workflow |
| MySQL | 8.0+ | Required when running backend source locally |
| RabbitMQ | 4.0 | `rabbitmq:4.0-management` is used by Compose |
| Keycloak | 26.0.7 | Used by the Compose configuration |

A 64-bit operating system with at least 8 GB of RAM is recommended because the complete stack runs several Java services and supporting containers.

## Get the Project

Clone the repository and enter its root directory:

```bash
git clone https://github.com/Kakadiya5986/salon.git
cd salon
```

The repository contains:

```text
salon/
├── backend (microservices)/
├── frontend/
├── UML_DIAGRAM.md
└── README.md
```

## Configuration and Security

Before running the backend, review the `application.yml` file in each backend service. The current source configuration expects:

- MySQL at `localhost:3306`
- RabbitMQ at `localhost:5672` with the default `guest` account
- Eureka at `http://localhost:8070/eureka/`
- Keycloak issuer keys at `http://localhost:8080/realms/master/protocol/openid-connect/certs`
- SMTP credentials for email notifications
- `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`, and `STRIPE_SECRET_KEY` environment variables for payment features

Do not publish real passwords, SMTP app passwords, or payment keys to GitHub. The repository currently contains credential values in some local configuration files. Rotate any credentials that have been exposed, move secrets to environment variables or a secrets manager, and remove them from tracked files before making the repository public.

## Run with Docker Compose

The Compose file is at `backend (microservices)/docker-compose/default/docker-compose.yml`. It starts RabbitMQ, MySQL containers, Keycloak, Eureka, the gateway, and prebuilt backend images referenced by the Compose file.

From the repository root:

```bash
cd "backend (microservices)/docker-compose/default"
docker compose pull
docker compose up -d
```

Check the running containers:

```bash
docker compose ps
```

Start the frontend in a separate terminal:

```bash
cd frontend
npm install
npm start
```

Open:

- Frontend: <http://localhost:3000>
- API gateway: <http://localhost:5000>
- Eureka dashboard: <http://localhost:8070>
- RabbitMQ management: <http://localhost:15672>
- Keycloak: <http://localhost:7080>

Stop the Compose stack:

```bash
docker compose down
```

To remove the database volumes as well, use `docker compose down -v`. This deletes local database data.

### Compose limitations

The Compose file uses prebuilt images such as `zosh/salon-user:v1` and `zosh/salon-gatewayserver:v1`; it does not build the Java source code. It also uses container-specific database hostnames and ports that differ from the local source `application.yml` files. Do not run the Compose backend services and the same backend services from source at the same time because their ports will conflict.

## Run the Backend from Source

Use this workflow when developing or changing the Java code.

### 1. Start dependencies

Start RabbitMQ:

```bash
docker run -d --name salon-rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:4.0-management
```

Start Keycloak on the port expected by the source gateway configuration:

```bash
docker run -d --name salon-keycloak -p 8080:8080 `
  -e KC_BOOTSTRAP_ADMIN_USERNAME=admin `
  -e KC_BOOTSTRAP_ADMIN_PASSWORD=admin `
  quay.io/keycloak/keycloak:26.0.7 start-dev
```

Install MySQL 8.0 or later and make it available on `localhost:3306`. Create the databases expected by the source services, or allow the configured `createDatabaseIfNotExist=true` option to create them:

```sql
CREATE DATABASE salon_userdb;
CREATE DATABASE salondb;
CREATE DATABASE salon_bookingdb;
CREATE DATABASE salon_categorydb;
CREATE DATABASE salon_servicedb;
CREATE DATABASE salon_paymentdb;
CREATE DATABASE salon_notificationdb;
CREATE DATABASE salon_reviewdb;
```

Update the database username and password in the service configuration files to match your local MySQL installation. Configure SMTP and payment credentials before using email or payment functionality.

### 2. Build the backend services

Run these commands from PowerShell in separate service directories, or build them one at a time:

```powershell
cd "backend (microservices)\eurekaserver"
.\mvnw.cmd clean package -DskipTests

cd ..\user-service
.\mvnw.cmd clean package -DskipTests

cd ..\salon
.\mvnw.cmd clean package -DskipTests

cd ..\booking
.\mvnw.cmd clean package -DskipTests

cd ..\category
.\mvnw.cmd clean package -DskipTests

cd ..\service-offering
.\mvnw.cmd clean package -DskipTests

cd ..\payment
.\mvnw.cmd clean package -DskipTests

cd ..\notifications
.\mvnw.cmd clean package -DskipTests

cd ..\review
.\mvnw.cmd clean package -DskipTests

cd ..\gateway-server
.\mvnw.cmd clean package -DskipTests
```

The Maven Wrapper downloads Maven 3.9.9 automatically, so a separate Maven installation is not required.

### 3. Start the backend services

Start each command in its own terminal. Start Eureka first, then the application services, and start the gateway last.

```powershell
cd "backend (microservices)\eurekaserver"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\user-service"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\salon"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\booking"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\category"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\service-offering"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\payment"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\notifications"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\review"
.\mvnw.cmd spring-boot:run
```

```powershell
cd "backend (microservices)\gateway-server"
.\mvnw.cmd spring-boot:run
```

When startup is complete, confirm that the services are registered at <http://localhost:8070> and that the gateway is available at <http://localhost:5000>.

## Run the Frontend

From the repository root:

```bash
cd frontend
npm install
npm start
```

Open <http://localhost:3000>. The frontend API client is configured in `frontend/src/config/api.js` and currently points to `http://localhost:5000`.

Available frontend commands:

```bash
npm start       # Start the development server
npm test        # Run the test runner
npm run build   # Create a production build in frontend/build
```

## Troubleshooting

### Port already in use

Stop the process using the port or change the relevant `server.port` value and update the frontend API URL or gateway configuration accordingly.

### Backend cannot connect to MySQL

Verify that MySQL is running on port `3306`, that the configured username and password are correct, and that the database names match the service `application.yml` files.

### Services do not appear in Eureka

Start Eureka before the other services and verify that each service points to `http://localhost:8070/eureka/`.

### Authentication fails

Verify that Keycloak is running on port `8080` for source mode, and that the gateway JWT `jwk-set-uri` points to the active Keycloak realm.

### Frontend API requests fail

Confirm that the gateway is running on port `5000`, then check `frontend/src/config/api.js` and the gateway CORS configuration.

## Tests

Run backend tests from an individual service directory:

```powershell
cd "backend (microservices)\booking"
.\mvnw.cmd test
```

Run frontend tests:

```bash
cd frontend
npm test
```

## Documentation

See [UML_DIAGRAM.md](UML_DIAGRAM.md) for domain and microservices architecture diagrams.
