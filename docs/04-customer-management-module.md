# Customer Management Module

## 1. Overview

The Customer Management module is responsible for managing customer information in the Bank Management System.

It provides functionality for customer registration, retrieving customer details, updating customer information, deactivating customers, and permanently deleting customer records when permitted by database relationships.

## 2. Technologies Used

- Java 21
- Spring Boot
- Spring MVC / REST API
- Spring Data JPA
- Hibernate ORM
- MySQL
- Jakarta Bean Validation
- Maven
- JUnit 5
- Mockito
- MockMvc

## 3. Architecture

The module follows a layered architecture:

1. Controller Layer – Handles incoming HTTP requests.
2. Service Layer – Implements customer business logic.
3. Repository Layer – Communicates with the database.
4. Entity Layer – Represents customer database records.
5. DTO Layer – Controls request and response data.
6. Exception Layer – Handles application exceptions centrally.

## 4. Customer Entity

The Customer entity contains:

| Field | Description |
|---|---|
| id | Primary key |
| customerNumber | Unique customer identifier |
| firstName | Customer's first name |
| lastName | Customer's last name |
| email | Unique email address |
| phone | Customer's phone number |
| dateOfBirth | Customer's date of birth |
| address | Customer's address |
| createdAt | Customer registration timestamp |
| updatedAt | Last update timestamp |
| status | ACTIVE or INACTIVE |

## 5. Implemented Functionalities

- Customer registration
- Retrieve all active customers
- Retrieve an active customer by ID
- Update customer information
- Soft deletion through customer deactivation
- Permanent deletion subject to database constraints
- Automatic customer number generation
- Duplicate email detection
- Input validation
- Centralized exception handling

## 6. Validation and Business Rules

- Customer number must follow the defined CUST identifier format.
- First and last names are mandatory.
- Email must be valid and unique.
- Phone number must contain exactly 10 digits when provided.
- Date of birth must be in the past.
- Address is mandatory.
- Inactive customers are excluded from normal customer retrieval.
- Customer identifiers and system-managed fields are not accepted from customer registration requests.

## 7. DTO Implementation

The module uses:

- CustomerRequestDTO – Receives customer registration and update information.
- CustomerResponseDTO – Returns customer information to API clients.
- CustomerMapper – Converts between entities and DTOs.

This prevents direct exposure of the database entity through the REST API.

## 8. Exception Handling

GlobalExceptionHandler provides centralized handling for:

- DuplicateResourceException – HTTP 409 Conflict
- ResourceNotFoundException – HTTP 404 Not Found
- MethodArgumentNotValidException – HTTP 400 Bad Request

Additional database-integrity and unexpected-exception handlers have also been reviewed.

## 9. Testing

The following testing activities were completed:

- CustomerService unit testing using JUnit 5 and Mockito.
- Customer API/controller testing using MockMvc.
- Registration and duplicate email scenarios.
- Customer retrieval and update scenarios.
- Customer deletion scenarios.
- Validation and exception scenarios.

**Test result:** All 15 Customer API/controller tests passed, along with the CustomerService unit tests.

## 10. Database

Database: `bank_management_system`

Table: `customers`

The table uses:
- BIGINT primary key with auto-increment.
- Unique customer number.
- Unique email.
- Customer status.
- Required customer information constraints.

## 11. Future Improvements

- Concurrency-safe customer number generation.
- More detailed database integrity exception handling.
- Authentication and authorization integration.
- Improved audit logging.

## 12. Module Status

Customer Management module implementation and initial testing are completed.

The module is ready for integration with the upcoming Account Management module, subject to final documentation and GitHub verification.
