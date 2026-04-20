***

# HBnB Technical Documentation

This document outlines the architecture, internal logic, and data flow for the HBnB application. The system is designed as a simplified vacation rental platform, emphasizing clean separation of concerns and maintainability.

## 1. High-Level Architecture

The application is built on a standard 3-tier architecture pattern. To simplify interactions between the user-facing endpoints and the underlying logic, the Presentation Layer acts as a Facade. It receives HTTP requests and delegates the heavy lifting to the services, without exposing how the data is processed or stored.

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
    Facade --> BusinessLogicLayer : Handles logic
    BusinessLogicLayer ..> PersistenceLayer : Database operations
```

## 2. Business Logic Layer

The core of the system revolves around four main entities. To avoid code duplication and ensure consistent auditing, all entities inherit from a single `BaseModel` that automatically handles unique identifiers (UUID4) and timestamps. 

A `User` can own multiple `Places` and write `Reviews`. A `Place` acts as the central resource: it belongs to an owner, contains a list of `Amenities` (many-to-many relationship), and receives `Reviews` from different users.

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

## 3. API Interaction Flow

The following sequence diagrams illustrate the step-by-step lifecycle of standard operations. In all cases, the API routes the payload to the appropriate Service class, which validates the business rules (e.g., verifying that a user exists before creating a place) before instructing the Database layer to persist the changes.

### User Registration

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

### Place Creation

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

### Review Submission

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

### Fetch Places

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
