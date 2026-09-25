# Salon Booking Project - UML Diagrams

## 1. Domain Model Class Diagram

```plantuml
@startuml DomainModel
!define BOLD(x) <b>x</b>

' Entities
class User {
    - id: Long
    - fullName: String
    - username: String
    - email: String
    - phone: String
    - role: UserRole
    - createdAt: LocalDateTime
    - updatedAt: LocalDateTime
}

class Salon {
    - id: Long
    - name: String
    - address: String
    - phoneNumber: String
    - email: String
    - city: String
    - openTime: LocalTime
    - closeTime: LocalTime
    - isOpen: Boolean
    - homeService: Boolean
    - active: Boolean
    - ownerId: Long
    - images: List<String>
}

class ServiceOffering {
    - id: Long
    - name: String
    - description: String
    - price: Double
    - duration: Integer (minutes)
    - salonId: Long
    - categoryId: Long
    - available: Boolean
    - image: String
}

class Category {
    - id: Long
    - name: String
    - image: String
    - salonId: Long
}

class Booking {
    - id: Long
    - salonId: Long
    - customerId: Long
    - startTime: LocalDateTime
    - endTime: LocalDateTime
    - serviceIds: Set<Long>
    - status: BookingStatus
    - totalPrice: Double
}

class PaymentOrder {
    - id: Long
    - amount: Double
    - status: PaymentOrderStatus
    - paymentMethod: PaymentMethod
    - paymentLinkId: String
    - userId: Long
    - bookingId: Long
    - salonId: Long
}

class Review {
    - id: Long
    - reviewText: String
    - rating: Double
    - salonId: Long
    - userId: Long
    - createdAt: LocalDateTime
}

class Notification {
    - id: Long
    - type: String
    - description: String
    - isRead: Boolean
    - userId: Long
    - bookingId: Long
    - salonId: Long
    - createdAt: LocalDateTime
}

class SalonReport {
    - salonId: Long
    - salonName: String
    - totalEarnings: Double
    - totalBookings: Long
    - cancelledBookings: Long
    - totalRefund: Double
}

' Enums
enum UserRole {
    ROLE_CUSTOMER
    ROLE_ADMIN
    ROLE_SALON_OWNER
}

enum BookingStatus {
    PENDING
    CONFIRMED
    CANCELLED
}

enum PaymentOrderStatus {
    PENDING
    SUCCESS
    FAILED
}

enum PaymentMethod {
    RAZORPAY
    STRIPE
}

' Relationships
User "1" --> "0..*" Salon : owns
User "1" --> "0..*" Booking : makes
User "1" --> "0..*" Review : writes
User "1" --> "0..*" Notification : receives

Salon "1" --> "0..*" Category : has
Salon "1" --> "0..*" ServiceOffering : offers
Salon "1" --> "0..*" Booking : receives
Salon "1" --> "0..*" Review : gets
Salon "1" --> "0..*" SalonReport : generates

Category "1" --> "0..*" ServiceOffering : contains

Booking "1" --> "0..*" ServiceOffering : includes
Booking "1" --> "1" PaymentOrder : has
Booking "1" --> "0..*" Notification : triggers

PaymentOrder "1" --> "1" User : paid_by
PaymentOrder "1" --> "1" Booking : for
PaymentOrder "1" --> "1" Salon : to

Review "1" --> "1" User : by
Review "1" --> "1" Salon : for

Notification "1" --> "1" User : to
Notification "1" --> "0..1" Booking : about
Notification "1" --> "0..1" Salon : about

User "1" --> "1" UserRole : has
Booking "1" --> "1" BookingStatus : status
PaymentOrder "1" --> "1" PaymentOrderStatus : status
PaymentOrder "1" --> "1" PaymentMethod : uses

@enduml
```

## 2. Microservices Architecture Diagram

```plantuml
@startuml MicroservicesArch
!define COMPONENT(x) rectangle x

package "Frontend" {
    COMPONENT(ReactApp) {
        rectangle "React Application" as frontend
    }
}

package "API Gateway" {
    COMPONENT(Gateway) {
        rectangle "Spring Cloud Gateway" as gateway
    }
}

package "Service Registry" {
    COMPONENT(EurekaServer) {
        rectangle "Eureka Server\n(Service Discovery)" as eureka
    }
}

package "Microservices" {
    COMPONENT(UserService) {
        rectangle "User Service" as user_svc
        rectangle "  - AuthService\n  - KeycloakUserService" as user_components
    }

    COMPONENT(SalonService) {
        rectangle "Salon Service" as salon_svc
        rectangle "  - SalonController\n  - SalonService\n  - SalonRepository" as salon_components
    }

    COMPONENT(BookingService) {
        rectangle "Booking Service" as booking_svc
        rectangle "  - BookingController\n  - BookingService\n  - PaymentOrder\n  - SalonReport" as booking_components
    }

    COMPONENT(ServiceOfferingService) {
        rectangle "Service Offering Service" as service_svc
        rectangle "  - ServiceOfferingController\n  - ServiceOfferingService" as service_components
    }

    COMPONENT(CategoryService) {
        rectangle "Category Service" as category_svc
        rectangle "  - CategoryController\n  - CategoryService" as category_components
    }

    COMPONENT(PaymentService) {
        rectangle "Payment Service" as payment_svc
        rectangle "  - PaymentController\n  - RazorpayService\n  - StripeService" as payment_components
    }

    COMPONENT(ReviewService) {
        rectangle "Review Service" as review_svc
        rectangle "  - ReviewController\n  - ReviewService" as review_components
    }

    COMPONENT(NotificationService) {
        rectangle "Notification Service" as notif_svc
        rectangle "  - NotificationController\n  - NotificationService\n  - BookingEventConsumer" as notif_components
    }
}

package "External Services" {
    rectangle "Razorpay\nPayment Gateway" as razorpay
    rectangle "Stripe\nPayment Gateway" as stripe
    rectangle "Keycloak\nAuth Provider" as keycloak
}

package "Data Layer" {
    database "User DB" as user_db
    database "Salon DB" as salon_db
    database "Booking DB" as booking_db
    database "Service DB" as service_db
    database "Category DB" as category_db
    database "Payment DB" as payment_db
    database "Review DB" as review_db
    database "Notification DB" as notif_db
}

package "Message Queue" {
    rectangle "Event Broker\n(Kafka/RabbitMQ)" as queue
}

' Relationships
frontend --> gateway : HTTP/REST
gateway --> user_svc : routes
gateway --> salon_svc : routes
gateway --> booking_svc : routes
gateway --> service_svc : routes
gateway --> category_svc : routes
gateway --> payment_svc : routes
gateway --> review_svc : routes
gateway --> notif_svc : routes

user_svc ..> eureka : registers
salon_svc ..> eureka : registers
booking_svc ..> eureka : registers
service_svc ..> eureka : registers
category_svc ..> eureka : registers
payment_svc ..> eureka : registers
review_svc ..> eureka : registers
notif_svc ..> eureka : registers

gateway ..> eureka : discovers services

booking_svc -.-> user_svc : UserFeignClient
booking_svc -.-> salon_svc : SalonFeignClient
booking_svc -.-> service_svc : ServiceOfferingFeignClient
booking_svc -.-> payment_svc : PaymentFeignClient

service_svc -.-> salon_svc : SalonFeignClient
service_svc -.-> category_svc : CategoryFeignClient

user_svc --> keycloak : authenticate
payment_svc --> razorpay : payment
payment_svc --> stripe : payment

user_svc --> user_db
salon_svc --> salon_db
booking_svc --> booking_db
service_svc --> service_db
category_svc --> category_db
payment_svc --> payment_db
review_svc --> review_db
notif_svc --> notif_db

notif_svc -.-> queue : consumes events
booking_svc -.-> queue : produces events

@enduml
```

## 3. Booking Flow Sequence Diagram

```plantuml
@startuml BookingFlow
participant Customer as customer
participant Frontend as frontend
participant Gateway as gateway
participant BookingService as booking
participant SalonService as salon
participant PaymentService as payment
participant NotificationService as notif

customer -> frontend: 1. Create Booking
frontend -> gateway: 2. POST /booking
gateway -> booking: 3. createBooking()
booking -> salon: 4. getAvailableSlots(FeignClient)
salon -> booking: 5. Return slots
booking -> payment: 6. createPaymentOrder(FeignClient)
payment -> payment: 7. Generate Razorpay Link
payment -> booking: 8. Return Payment Link
booking -> frontend: 9. Return Payment Link
frontend -> customer: 10. Display Payment Form

customer -> frontend: 11. Pay via Razorpay
frontend -> payment: 12. Webhook: paymentSuccess()
payment -> booking: 13. Update Booking Status to CONFIRMED
booking -> booking: 14. generateSalonReport()
booking -> notif: 15. Create Notification Event (Async)
notif -> notif: 16. Notify Customer & Salon Owner
notif -> customer: 17. Notification: Booking Confirmed

@enduml
```

## 4. Component Dependencies

| Service | Depends On | Communication |
|---------|-----------|----------------|
| **Booking** | User, Salon, ServiceOffering, Payment | Feign Clients |
| **ServiceOffering** | Salon, Category | Feign Clients |
| **Payment** | External (Razorpay, Stripe) | HTTP |
| **Notification** | Booking Events | Message Queue |
| **Review** | User, Salon | Direct References |
| **Category** | Salon | Direct References |
| **User** | Keycloak | OAuth2 |
| **Salon** | None (Independent) | - |
| **Gateway** | Eureka, All Services | Service Discovery |

## 5. Technology Stack

- **Backend Framework**: Spring Boot + Spring Cloud
- **Service Discovery**: Spring Cloud Eureka
- **API Gateway**: Spring Cloud Gateway
- **Authentication**: Keycloak + OAuth2
- **Inter-service Communication**: Feign Clients
- **Payment Processing**: Razorpay & Stripe APIs
- **Messaging**: Kafka/RabbitMQ (for async events)
- **Database**: Per-service databases (Database per Service pattern)
- **Containerization**: Docker (docker-compose.yml for orchestration)

## 6. Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Docker Compose                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │   User DB   │  │  Booking DB │  │  Payment DB │        │
│  │ (PostgreSQL)│  │(PostgreSQL) │  │(PostgreSQL) │        │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘        │
│         │                │                │                │
│  ┌──────────────────────────────────────────────────┐      │
│  │           Service Containers                    │      │
│  ├──────────────────────────────────────────────────┤      │
│  │ UserService  SalonService  BookingService       │      │
│  │ PaymentService  NotificationService  ...        │      │
│  └──────────────────────────────────────────────────┘      │
│                          │                                 │
│  ┌──────────────────────────────────────────────────┐      │
│  │  Gateway (Port 8080)                             │      │
│  └──────────────────────────────────────────────────┘      │
│                          │                                 │
└──────────────────────────┼─────────────────────────────────┘
                           │
                   ┌───────┴────────┐
                   │                │
            ┌──────────────┐  ┌──────────────┐
            │   Frontend   │  │  Keycloak    │
            │   (React)    │  │  (OAuth2)    │
            └──────────────┘  └──────────────┘
```

