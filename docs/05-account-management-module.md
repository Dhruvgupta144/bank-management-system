# Account Management Module

## 1. Module Overview

The Account Management module is responsible for creating, maintaining, retrieving, and managing bank accounts associated with registered customers.

It is a core module of the Bank Management System. It maintains account information and controls the account lifecycle, including account creation, activation, deactivation, and closure.

Each account belongs to one registered customer, while a customer can own multiple accounts.

The module is developed using Java, Spring Boot, Spring Data JPA, Hibernate, and MySQL.

---

## 2. Objectives

The main objectives of this module are:

- Create bank accounts for registered customers.
- Generate unique account numbers.
- Associate accounts with existing customers.
- Maintain account types and account statuses.
- Retrieve individual account details.
- Retrieve accounts based on their status.
- Retrieve all accounts associated with a customer.
- Activate and deactivate accounts.
- Close accounts while preserving their records.
- Maintain account creation, update, opening, and closure timestamps.
- Enforce banking business rules and database constraints.

---

## 3. Scope of the Module

### Included Features

The initial version of the Account Management module includes:

1. Open a new bank account.
2. Retrieve an active account by account number.
3. Retrieve a closed account by account number.
4. Retrieve all active accounts.
5. Retrieve all closed accounts.
6. Retrieve all accounts belonging to a customer.
7. Activate an inactive account.
8. Deactivate an active account.
9. Close an account.

### Excluded Features

The following operations are handled by other modules or are outside the current scope:

- Deposits and withdrawals.
- Fund transfers between accounts.
- Transaction history and account statements.
- Interest calculation.
- Loan management.
- Credit and debit card management.
- ATM and UPI integration.
- External banking network integration.
- Direct modification of account balances through an update API.

Deposits, withdrawals, and transfers will be implemented in the Banking Transactions module.

---

## 4. Account Types

The system supports two account types:

| Account Type | Description |
|---|---|
| SAVINGS | A savings account intended for individual customers. |
| CURRENT | A current account intended for business and other eligible banking purposes. |

The account type is selected when the account is created and cannot be changed through a general account update operation.

### Account Ownership Rules

- A customer can have multiple accounts.
- A customer can have one active SAVINGS account.
- A customer can have one active CURRENT account.
- A customer may open another account of the same type after the previous one is closed.
- Previous account records must be preserved.

---

## 5. Account Status Management

The system supports three account statuses:

| Status | Description |
|---|---|
| ACTIVE | The account is active and eligible for permitted banking operations. |
| INACTIVE | The account is temporarily deactivated. |
| CLOSED | The account is permanently closed and cannot be reactivated. |

### Account Lifecycle

The account lifecycle follows these transitions:

```text
                 ACTIVATE
        ┌────────────────────────┐
        │                        ▼
    INACTIVE ◄─────────────── ACTIVE
        │                        │
        │                        │
        └──────────┬─────────────┘
                   │
                   ▼
                 CLOSED
```

Both ACTIVE and INACTIVE accounts can be closed, provided the account balance is zero.

A CLOSED account cannot be activated or deactivated again.

### Status Transition Rules

| Current Status | Operation | New Status | Allowed |
|---|---|---|---|
| ACTIVE | Deactivate | INACTIVE | Yes |
| INACTIVE | Activate | ACTIVE | Yes |
| ACTIVE | Close | CLOSED | Yes, if balance is zero |
| INACTIVE | Close | CLOSED | Yes, if balance is zero |
| CLOSED | Activate | — | No |
| CLOSED | Deactivate | — | No |
| CLOSED | Close | — | No |

---

## 6. Database Design

### Accounts Table

The `accounts` table stores account-related information.

| Column | Data Type | Constraints | Description |
|---|---|---|---|
| id | BIGINT | Primary Key, Auto Increment | Internal database identifier |
| account_number | VARCHAR(20) | NOT NULL, UNIQUE | Unique account number |
| customer_id | BIGINT | NOT NULL, Foreign Key | References the customer |
| account_type | VARCHAR(20) | NOT NULL | SAVINGS or CURRENT |
| balance | DECIMAL(15,2) | NOT NULL, DEFAULT 0.00 | Current account balance |
| currency | VARCHAR(3) | NOT NULL, DEFAULT INR | Account currency |
| status | VARCHAR(20) | NOT NULL, DEFAULT ACTIVE | Current account status |
| opened_at | TIMESTAMP | NOT NULL | Account opening timestamp |
| closed_at | TIMESTAMP | NULL | Account closure timestamp |
| created_at | TIMESTAMP | NOT NULL | Record creation timestamp |
| updated_at | TIMESTAMP | NOT NULL | Last record update timestamp |

### Database Relationship

The relationship between customers and accounts is:

```text
CUSTOMERS
    |
    | 1
    |
    | Many
    ▼
ACCOUNTS
```

- One customer can have multiple accounts.
- Each account belongs to exactly one customer.
- The `customer_id` column establishes the foreign key relationship.
- An account cannot be created without an existing customer.

### Account Number Generation

Account numbers follow this format:

```text
ACC + 9 Numeric Digits
```

Example:

```text
ACC000000001
ACC000000002
ACC000000003
```

A separate account number sequence table is used to generate sequential identifiers.

The account number is unique and is independent of the internal database ID of the account.

### Database Constraints

The database should enforce the following constraints:

- Unique account numbers.
- Valid customer foreign key.
- Nonnegative account balance.
- Valid account types.
- Valid account statuses.
- Only one ACTIVE account of a given type per customer.

The last rule can be enforced in MySQL using a generated column and a unique index on `(customer_id, active_account_type)`, where the generated column contains the account type only for ACTIVE accounts and NULL otherwise.

---

## 7. Major Business Rules

The Account Management module follows these rules:

1. An account must belong to an existing customer.
2. Only ACTIVE customers can open new accounts.
3. Every account must have a unique account number.
4. Every newly created account starts with a balance of INR 0.00.
5. Every newly created account receives ACTIVE status.
6. A customer can have only one ACTIVE account of each account type.
7. A customer may open another account of the same type after the previous account is closed.
8. Account numbers cannot be changed after creation.
9. Account ownership cannot be changed through a general account update operation.
10. Account type cannot be changed after creation.
11. Account balance cannot become negative.
12. Only INACTIVE accounts can be activated.
13. Only ACTIVE accounts can be deactivated.
14. ACTIVE or INACTIVE accounts can be closed.
15. An account can only be closed when its balance is exactly zero.
16. CLOSED accounts cannot be reopened.
17. Deactivation and closure must not delete account records.
18. Account creation and status changes must preserve account history.
19. Financial operations must reject accounts that are not ACTIVE.
20. Restricted operations must be accessible only to authorized users.

---

## 8. API Design

The Account Management module exposes REST APIs under:

```text
/api/accounts
```

### 8.1 Open Account

| Property | Value |
|---|---|
| HTTP Method | POST |
| Endpoint | `/api/accounts` |
| Description | Creates a new account for an existing customer. |
| Success Response | 201 Created |

Request body:

```json
{
    "customerNumber": "CUST1001",
    "accountType": "SAVINGS"
}
```

The system validates the customer, checks account eligibility, generates an account number, and creates the account with zero balance and ACTIVE status.

### 8.2 Retrieve Active Account

| Property | Value |
|---|---|
| HTTP Method | GET |
| Endpoint | `/api/accounts/active/{accountNumber}` |
| Description | Retrieves an individual active account. |
| Success Response | 200 OK |

### 8.3 Retrieve Closed Account

| Property | Value |
|---|---|
| HTTP Method | GET |
| Endpoint | `/api/accounts/closed/{accountNumber}` |
| Description | Retrieves an individual closed account. |
| Success Response | 200 OK |

### 8.4 Retrieve All Active Accounts

| Property | Value |
|---|---|
| HTTP Method | GET |
| Endpoint | `/api/accounts/active` |
| Description | Retrieves all active accounts. |
| Success Response | 200 OK |

### 8.5 Retrieve All Closed Accounts

| Property | Value |
|---|---|
| HTTP Method | GET |
| Endpoint | `/api/accounts/closed` |
| Description | Retrieves all closed accounts. |
| Success Response | 200 OK |

### 8.6 Retrieve Customer Accounts

| Property | Value |
|---|---|
| HTTP Method | GET |
| Endpoint | `/api/accounts/customer/{customerNumber}` |
| Description | Retrieves all accounts associated with a customer. |
| Success Response | 200 OK |

The response includes accounts with ACTIVE, INACTIVE, and CLOSED statuses.

### 8.7 Activate Account

| Property | Value |
|---|---|
| HTTP Method | PATCH |
| Endpoint | `/api/accounts/{accountNumber}/activate` |
| Description | Activates an inactive account. |
| Success Response | 200 OK |

### 8.8 Deactivate Account

| Property | Value |
|---|---|
| HTTP Method | PATCH |
| Endpoint | `/api/accounts/{accountNumber}/deactivate` |
| Description | Temporarily deactivates an active account. |
| Success Response | 200 OK |

### 8.9 Close Account

| Property | Value |
|---|---|
| HTTP Method | PATCH |
| Endpoint | `/api/accounts/{accountNumber}/close` |
| Description | Permanently closes an account with zero balance. |
| Success Response | 200 OK |

---

## 9. Request and Response DTOs

The module uses Data Transfer Objects (DTOs) to separate API data from database entities.

### AccountRequestDTO

The account creation request contains:

- customerNumber
- accountType

The customer is identified using the business customer number instead of exposing the internal database ID.

### AccountResponseDTO

The account response contains:

- accountNumber
- customer details
- accountType
- balance
- currency
- status
- openedAt
- closedAt
- createdAt
- updatedAt

Customer information is included through a nested `CustomerResponseDTO`.

Internal database identifiers are not exposed through the account response.

### AccountMapper

The mapper is responsible for converting:

- AccountRequestDTO into Account.
- Account entity into AccountResponseDTO.

Customer lookup and association are handled by the service layer because the request contains a customer number rather than a Customer entity.

---

## 10. Application Architecture

The module follows a layered architecture.

```text
             CLIENT
          (Postman / UI)
                 |
                 ▼
          ACCOUNT CONTROLLER
                 |
                 ▼
           ACCOUNT SERVICE
                 |
          ┌──────┴──────┐
          ▼             ▼
  ACCOUNT REPOSITORY  CUSTOMER REPOSITORY
          |             |
          └──────┬──────┘
                 ▼
              MYSQL
```

### Controller Layer

The controller handles HTTP requests and responses.

Responsibilities:

- Define REST endpoints.
- Receive request data.
- Apply request validation using `@Valid`.
- Call the service layer.
- Return appropriate HTTP responses.

### Service Layer

The service layer contains business logic.

Responsibilities:

- Validate customer eligibility.
- Check account ownership relationships.
- Generate account numbers.
- Apply account creation rules.
- Enforce account status transitions.
- Check account balance before closure.
- Convert entities into response DTOs.
- Manage transactional business operations.

### Repository Layer

The repository layer communicates with the database using Spring Data JPA.

It extends `JpaRepository` and provides database operations such as:

- Saving accounts.
- Finding accounts by account number.
- Finding accounts by status.
- Checking whether an active account of a given type already exists.
- Finding all accounts belonging to a customer.

### Entity Layer

The Account entity represents the `accounts` database table.

It maintains:

- Account identity.
- Customer relationship.
- Account type.
- Balance and currency.
- Account status.
- Account timestamps.

### DTO and Mapper Layer

DTOs define the data exchanged through REST APIs, while the mapper converts between entities and DTOs.

---

## 11. Validation and Exception Handling

The module uses Jakarta Bean Validation and centralized exception handling.

### Request Validation

Examples of validation rules:

- Customer number is required.
- Customer number must follow the expected format.
- Account type must not be null.
- Account type must be a supported enum value.

### Exception Handling

The module uses custom exceptions and a global exception handler.

| Exception / Condition | HTTP Status | Description |
|---|---|---|
| ResourceNotFoundException | 404 Not Found | Requested customer or account does not exist. |
| BusinessRuleException | 409 Conflict | An operation violates a banking business rule. |
| DataIntegrityViolationException | 409 Conflict | A database constraint is violated. |
| MethodArgumentNotValidException | 400 Bad Request | Request validation fails. |
| Invalid enum value | 400 Bad Request | Unsupported account type or status value. |
| Unexpected exception | 500 Internal Server Error | Unexpected server-side failure. |

Error responses should provide a clear message without exposing sensitive database or internal implementation details.

---

## 12. Transaction Management

The service layer uses Spring's `@Transactional` annotation to manage database transactions.

Transaction management is important when a business operation involves multiple database operations.

For example, opening an account involves:

1. Generating an account number using the sequence table.
2. Saving the new account.
3. Returning the created account details.

The database changes should be rolled back if the operation fails.

Database-generated sequence values may still have gaps after rollback. Account numbers do not need to be gap-free; they need to be unique.

---

## 13. Testing Strategy

The module should be tested at multiple levels.

### Unit Testing

JUnit 5 and Mockito are used to test service-layer business logic.

Important test cases:

- Successful account creation.
- Account creation for a nonexistent customer.
- Account creation for an inactive customer.
- Duplicate active account type rejection.
- Successful account activation.
- Successful account deactivation.
- Account closure with zero balance.
- Rejection of account closure with positive balance.
- Rejection of activation for a closed account.
- Rejection of repeated status transitions.

### Controller Testing

MockMvc is used to test REST endpoints.

Important test cases:

- Valid account creation request.
- Invalid customer number.
- Missing account type.
- Retrieving an existing account.
- Retrieving an account that does not exist.
- Retrieving accounts by status.
- Retrieving customer accounts.
- Activating, deactivating, and closing accounts.

### Database Testing

The database should be checked to verify:

- Account numbers are unique.
- Customer foreign keys are valid.
- Initial balances are zero.
- Account statuses are stored correctly.
- Account timestamps are maintained.
- Account closure preserves the account record.
- Database constraints prevent invalid account data.

### API Testing

Postman is used for manual API testing.

Both successful and unsuccessful scenarios should be verified.

---

## 14. Security Considerations

The current module is developed before the complete authentication and authorization implementation.

In the planned security implementation:

- Only authorized employees or administrators should perform restricted account management operations.
- Customers should only access their own accounts.
- A customer must not be allowed to access another customer's account by changing the customer number in a request.
- Account status changes must be protected by role-based authorization.
- Sensitive customer information must not be exposed unnecessarily.

When Spring Security and JWT authentication are implemented, customer identity should be derived from the authenticated user rather than trusted directly from client-supplied customer identifiers.

---

## 15. Future Enhancements

The following features may be added in future versions:

- Account summary and dashboard.
- Account statement generation.
- Transaction history.
- Monthly and yearly account reports.
- Account search and advanced filtering.
- Pagination and sorting.
- Account notifications.
- Interest calculation for savings accounts.
- Audit logs for account status changes.
- Role-based account management.
- Automated account maintenance.

---

## 16. Conclusion

The Account Management module provides the foundation for managing bank accounts in the Bank Management System.

It supports account creation, retrieval, ownership association, and account lifecycle management while enforcing important banking rules.

The layered architecture separates HTTP handling, business logic, database access, and data transfer responsibilities.

This module also provides the foundation for upcoming Banking Transactions functionality, where deposits, withdrawals, transfer, and transaction history will be implemented.

The module is designed to preserve account records and maintain data integrity throughout the account lifecycle.