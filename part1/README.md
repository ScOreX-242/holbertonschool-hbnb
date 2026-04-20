---

# 📘 HBnB Technical Documentation

---

## 1. Introduction

This document provides a comprehensive technical overview of the HBnB application architecture, design, and interaction flow. It serves as a blueprint to guide the implementation phase and ensure consistency across the system.

HBnB is a platform that allows users to:

* Register and manage accounts
* Create and manage place listings
* Submit reviews
* Browse available places

The document includes:

* High-level architecture (package diagram)
* Business logic (class diagram)
* API interaction flow (sequence diagrams)

---

## 2. High-Level Architecture

###  Overview

The system follows a **layered architecture** combined with a **Facade pattern**:

* **Presentation Layer** → API (entry point)
* **Business Logic Layer** → Services & Models
* **Persistence Layer** → Database

---

###  High-Level Package Diagram

```mermaid
classDiagram
    direction TB

    class PresentationLayer {
        +API Endpoints
        +Services
    }

    class BusinessLogicLayer {
        +User
        +Place
        +Review
        +Amenity
    }

    class PersistenceLayer {
        +UserRepository
        +PlaceRepository
        +ReviewRepository
        +AmenityRepository
    }

    class Facade {
        +createUser()
        +getPlaces()
        +addReview()
        +addAmenity()
    }

    PresentationLayer ..> Facade : Uses
    Facade --> BusinessLogicLayer : Handles business logic
    BusinessLogicLayer ..> PersistenceLayer : Database operations
```

---

###  Explanation

* **API** acts as a Facade — it hides internal complexity
* Services contain business rules
* Models represent core entities
* Database handles persistence

---

## 3. Business Logic Layer

###  Class Diagram

```mermaid
classDiagram

    class BaseModel {
        +UUID4 id
        +DateTime created_at
        +DateTime updated_at
        +save()
        +update()
        +delete()
    }

    class User {
        +String first_name
        +String last_name
        +String email
        +String password
        +Boolean is_admin
        +register()
        +update_profile()
    }

    class Place {
        +String title
        +String description
        +Float price
        +Float latitude
        +Float longitude
        +UUID4 owner_id
        +create()
        +update()
    }

    class Review {
        +Int rating
        +String comment
        +UUID4 place_id
        +UUID4 user_id
        +post()
    }

    class Amenity {
        +String name
        +String description
        +create()
    }

    User --|> BaseModel : Inherits
    Place --|> BaseModel : Inherits
    Review --|> BaseModel : Inherits
    Amenity --|> BaseModel : Inherits

    User "1" --> "0..*" Place : Owns
    User "1" --> "0..*" Review : Writes
    Place "1" --> "0..*" Review : Has
    Place "0..*" --> "0..*" Amenity : Includes
```

---

###  Explanation

#### 🔹 User

* Handles authentication and identity
* Can create places and reviews

#### 🔹 Place

* Represents listings
* Linked to a User (owner)

#### 🔹 Review

* Feedback left by users
* Linked to both Place and User

---

###  Design Decisions

* Separation of concerns → each entity has a clear responsibility
* Relationships reflect real-world logic
* Scalable structure for adding features (e.g., bookings)

---

## 4. API Interaction Flow

This section describes how different layers interact when processing API requests.

---

## 🔹 4.1 User Registration

```mermaid
sequenceDiagram
participant User
participant API
participant UserService
participant Database

User->>API: POST /users
API->>UserService: validateUser(data)
UserService->>Database: insertUser(user)
Database-->>UserService: success
UserService-->>API: response
API-->>User: 201 Created
```

###  Explanation

* User sends registration request
* Data validated in service layer
* Stored in database
* Response returned

---

## 🔹 4.2 Place Creation

```mermaid
sequenceDiagram
participant User
participant API
participant PlaceService
participant Database

User->>API: POST /places
API->>PlaceService: createPlace(data)
PlaceService->>Database: verifyUser(owner_id)
PlaceService->>Database: insertPlace(place)
Database-->>PlaceService: success
PlaceService-->>API: response
API-->>User: 201 Created
```

###  Explanation

* User creates a listing
* Ownership is validated
* Place is saved

---

## 🔹 4.3 Review Submission

```mermaid
sequenceDiagram
participant User
participant API
participant ReviewService
participant Database

User->>API: POST /reviews
API->>ReviewService: submitReview(data)
ReviewService->>Database: checkPlace(place_id)
ReviewService->>Database: insertReview(review)
Database-->>ReviewService: success
ReviewService-->>API: response
API-->>User: 201 Created
```

###  Explanation

* User submits review
* System checks if place exists
* Review is saved

---

## 🔹 4.4 Fetch Places

```mermaid
sequenceDiagram
participant User
participant API
participant PlaceService
participant Database

User->>API: GET /places
API->>PlaceService: getPlaces(filters)
PlaceService->>Database: queryPlaces()
Database-->>PlaceService: places list
PlaceService-->>API: data
API-->>User: 200 OK
```

###  Explanation

* User requests data
* Filters applied
* Results returned

---

## 5. Overall Architecture Summary

###  Key Points

* **Layered Design**

  * Clear separation between API, logic, and data

* **Facade Pattern**

  * API simplifies interactions

* **Scalability**

  * Easy to extend services and models

* **Maintainability**

  * Clean structure reduces complexity

---

## 6. Conclusion

This document provides a structured and complete overview of the HBnB system design. It ensures that:

* All components are clearly defined
* Interactions between layers are well understood
* Developers can use it as a reference during implementation

---
