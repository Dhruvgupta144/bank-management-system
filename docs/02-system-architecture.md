# Bank Management System — System Architecture

## 1. Architecture Overview

The Bank Management System will use a layered architecture to separate the user interface, application logic, business rules, and database operations.

The architecture will be designed to provide:

- Separation of responsibilities
- Maintainability
- Security
- Scalability
- Testability
- Reusability
- Clear communication between application layers

The initial system will follow a client-server architecture using a web-based application.

### High-Level Architecture

```text
                    User
                     |
                     ↓
              Web Browser
                     |
                     ↓
              REST API Layer
                     |
                     ↓
             Controller Layer
                     |
                     ↓
              Service Layer
                     |
                     ↓
             Repository Layer
                     |
                     ↓
               MySQL Database
