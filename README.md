# Spring Boot Invoice Fullstack

<em>A full-stack electronic invoicing system combining Spring Boot REST API with a Vanilla JavaScript SPA frontend. Features Spring Security authentication, comprehensive invoice management, and PDF generation. This is an enhanced version of [springboot-invoice-management](https://github.com/isaacmendezr/springboot-invoice-management) with modern fullstack architecture.</em>

---

## Table of Contents

- [Spring Boot Invoice Fullstack](#spring-boot-invoice-fullstack)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Architecture](#architecture)
  - [Features](#features)
  - [Technology Stack](#technology-stack)
  - [Database Model](#database-model)
  - [API Endpoints](#api-endpoints)
  - [Project Structure](#project-structure)
  - [Getting Started](#getting-started)
    - [Prerequisites](#prerequisites)
    - [Installation](#installation)
    - [Database Setup](#database-setup)
    - [Configuration](#configuration)
    - [Usage](#usage)
  - [Security](#security)
  - [Frontend](#frontend)
  - [License](#license)
  - [Contact](#contact)

---

## Overview

**Spring Boot Invoice Fullstack** is a comprehensive electronic invoicing system designed for small and medium-sized businesses. This fullstack application extends the original [springboot-invoice-management](https://github.com/isaacmendezr/springboot-invoice-management) by implementing a modern architecture with a Spring Boot REST API backend and a Vanilla JavaScript SPA frontend.

Key improvements over the previous version:
- **Fullstack Architecture**: Complete separation of concerns with REST API backend and JavaScript SPA frontend
- **Spring Security**: Robust authentication and authorization with session-based security
- **Modern Frontend**: Vanilla JavaScript with Fetch API for asynchronous communication
- **Enhanced Security**: BCrypt password encryption and role-based access control
- **DTOs**: Proper data transfer objects for clean API contracts

The application supports two main user roles:
- **Administrators**: Manage supplier accounts and approval workflows
- **Suppliers**: Manage clients, products, and generate invoices with PDF export

---

## Architecture

**Backend:**
- **Framework:** Spring Boot 3.2.5
- **Language:** Java 21
- **Database:** MySQL 8.0+
- **ORM:** Spring Data JPA (Hibernate)
- **Design Pattern:** Layered Architecture (Presentation → Service → Repository)
- **API Style:** REST API with JSON
- **Authentication:** Spring Security with session-based auth
- **Packaging:** WAR (Web Application Archive)

**Frontend:**
- **Type:** Single Page Application (SPA)
- **Language:** Vanilla JavaScript (ES6+)
- **HTTP Client:** Fetch API
- **Styling:** CSS3 with Bootstrap components
- **State Management:** Global state object for user authentication

The application follows a three-tier architecture:
1. **Presentation Layer** (`presentation`): REST Controllers exposing JSON endpoints
2. **Business Logic Layer** (`logic`): Service classes with transactional operations and domain entities
3. **Data Access Layer** (`data`): Spring Data JPA repositories

---

## Features

| Category | Description |
| :-------- | :----------- |
| 🔐 **Authentication & Authorization** | Spring Security integration with role-based access control (ADMINISTRATOR/PROVEEDOR), session management, BCrypt password encryption, and user state validation (ACTIVO/INACTIVO/PENDIENTE) |
| 👥 **User Management** | Supplier self-registration with pending status, administrator approval workflow, profile updates, and secure password handling |
| 📊 **Client Management** | Full CRUD operations via REST API, search by name functionality, client assignment per supplier with validation |
| 📦 **Product Catalog** | Product management per supplier with name, description, and pricing, search capabilities, REST endpoints for all operations |
| 🧾 **Invoice Generation** | Real-time invoice creation with automatic calculations, dynamic item quantity adjustments, subtotal and total computation, client and product validation |
| 📄 **PDF Export** | Professional invoice PDF generation using iText7 library, complete invoice details with line items, client information, and totals |
| 🎯 **Admin Panel** | Supplier activation/deactivation endpoints, sortable supplier lists, approval workflow management, role-based access restrictions |
| 🔍 **REST API** | Complete RESTful endpoints for all operations, JSON request/response format, proper HTTP status codes, CORS support for frontend integration |
| 🛡️ **Security Features** | Custom UserDetailsService implementation, lazy loading with Jackson filters, CSRF protection configuration, session-based authentication |

---

## Technology Stack

**Core Dependencies:**
- **Spring Boot Starter Web** - REST API framework and embedded Tomcat server
- **Spring Boot Starter Data JPA** - Database operations and ORM with Hibernate
- **Spring Boot Starter Security** - Authentication and authorization framework
- **Spring Boot Starter Validation** - Jakarta Bean Validation (JSR-380)
- **MySQL Connector** - MySQL database driver
- **iText7 Core (7.1.0)** - PDF generation and manipulation library
- **Spring Boot DevTools** - Hot reload and development utilities

**Configuration:**
- **Server Port:** 8080 (default)
- **Database URL:** `jdbc:mysql://localhost:3306/Facturacion`
- **JPA Show SQL:** Enabled for debugging
- **Naming Strategy:** PhysicalNamingStrategyStandardImpl (preserves exact column names)
- **Open-in-View:** Disabled (false) for better performance
- **CSRF:** Disabled for REST API
- **Session Management:** Stateful sessions with HTTP-only cookies

---

## Database Model

The application uses a relational MySQL database with the following entities:

**Core Entities:**
- **Usuario (User)**: Authentication and authorization
  - Fields: id (PK), nombreUsuario (unique), contrasena (BCrypt), estado (ENUM), rol (ENUM)
  - States: ACTIVO, INACTIVO, PENDIENTE
  - Roles: ADMINISTRADOR, PROVEEDOR

- **Proveedor (Supplier)**: Supplier information with 1:1 relationship to Usuario
  - Fields: id (PK, tax ID), usuarioID (FK), nombre, correoElectronico (unique), numeroTelefono (unique), direccion
  - Validations: Email format, phone format, size constraints

- **Cliente (Client)**: Client records linked to suppliers
  - Fields: id (PK, tax ID), nombre, correoElectronico (unique), numeroTelefono (unique), direccion, proveedorID (FK)
  - N:1 relationship with Proveedor

- **Producto (Product)**: Product catalog per supplier
  - Fields: id (PK, auto-increment), nombre, descripcion, precio (DECIMAL), proveedorID (FK)
  - N:1 relationship with Proveedor

- **Factura (Invoice)**: Invoice headers
  - Fields: id (PK, auto-increment), fecha (DATE), total (DECIMAL), proveedorID (FK), clienteID (FK)
  - N:1 relationships with Proveedor and Cliente

- **DetalleFactura (Invoice Detail)**: Line items with dynamic subtotal calculation
  - Fields: id (PK, auto-increment), facturaID (FK), productoID (FK), cantidad, precioUnitario (DECIMAL), subtotal (Transient - calculated)
  - N:1 relationships with Factura and Producto

**Relationships:**
- Usuario ↔ Proveedor: One-to-One (Bidirectional)
- Proveedor → Cliente: One-to-Many
- Proveedor → Producto: One-to-Many
- Proveedor → Factura: One-to-Many
- Cliente → Factura: One-to-Many
- Factura → DetalleFactura: One-to-Many
- Producto → DetalleFactura: One-to-Many

---

## API Endpoints

**Authentication Endpoints:**
| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/login/login` | Authenticate user and create session | Public |
| POST | `/api/login/logout` | Destroy session and logout | Authenticated |
| GET | `/api/login/current-user` | Get current authenticated user | Authenticated |

**Registration & Profile:**
| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| POST | `/api/registro/register` | Register new supplier (status: PENDIENTE) | Public |
| GET | `/api/perfil/get` | Get supplier profile | Supplier |
| POST | `/api/perfil/update` | Update supplier profile | Supplier |

**Supplier Management (Admin):**
| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/api/proveedores/listar` | List all suppliers with status | Admin |
| POST | `/api/proveedores/activar/{id}` | Activate supplier account | Admin |
| POST | `/api/proveedores/desactivar/{id}` | Deactivate supplier account | Admin |

**Client Management:**
| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/api/clientes` | List all clients | Supplier/Admin |
| GET | `/api/clientes/{id}` | Get client by ID | Supplier/Admin |
| GET | `/api/clientes/search?nombre={name}` | Search clients by name | Supplier/Admin |
| POST | `/api/clientes` | Create new client | Supplier/Admin |
| PUT | `/api/clientes/{id}` | Update client | Supplier/Admin |

**Product Management:**
| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/api/productos` | List all products | Supplier/Admin |
| GET | `/api/productos/{id}` | Get product by ID | Supplier/Admin |
| GET | `/api/productos/search?nombre={name}` | Search products by name | Supplier/Admin |
| POST | `/api/productos` | Create new product | Supplier/Admin |
| PUT | `/api/productos/{id}` | Update product | Supplier/Admin |

**Invoice Operations:**
| Method | Endpoint | Description | Access |
|--------|----------|-------------|--------|
| GET | `/api/facturar/search?id={clientId}` | Get client for invoicing | Supplier |
| GET | `/api/facturar/searchProveedor?id={userId}` | Get supplier by user ID | Supplier |
| GET | `/api/facturar/searchProducto?id={productId}` | Get product with detail template | Supplier |
| POST | `/api/facturar` | Create invoice with details | Supplier |
| GET | `/api/facturas?id={supplierId}` | List invoices by supplier | Supplier |
| GET | `/api/facturas/{id}/pdf` | Generate and download PDF | Supplier |

---

## Project Structure

```sh
springboot-invoice-fullstack/
├── DB/
│   ├── Facturacion_Schema.sql           # Database schema definition
│   └── Facturacion_Data.sql             # Sample data with encrypted passwords
├── FacturacionCSR/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/eif209/facturacioncsr/
│   │   │   │   ├── FacturacionCsrApplication.java    # Main app + Security config
│   │   │   │   ├── ServletInitializer.java           # WAR deployment config
│   │   │   │   ├── PasswordUpdater.java              # Utility for password encryption
│   │   │   │   ├── data/                             # Repositories
│   │   │   │   │   ├── ClienteRepository.java
│   │   │   │   │   ├── DetallefacturaRepository.java
│   │   │   │   │   ├── FacturaRepository.java
│   │   │   │   │   ├── ProductoRepository.java
│   │   │   │   │   ├── ProveedorRepository.java
│   │   │   │   │   └── UsuarioRepository.java
│   │   │   │   ├── logic/                            # Domain layer
│   │   │   │   │   ├── Cliente.java                  # Client entity
│   │   │   │   │   ├── Detallefactura.java          # Invoice detail entity
│   │   │   │   │   ├── Factura.java                 # Invoice entity
│   │   │   │   │   ├── Producto.java                # Product entity
│   │   │   │   │   ├── Proveedor.java               # Supplier entity
│   │   │   │   │   ├── Usuario.java                 # User entity
│   │   │   │   │   ├── Service.java                 # Business logic service
│   │   │   │   │   ├── GuardarFactura.java          # Invoice DTO helper
│   │   │   │   │   ├── LazyFieldsFilter.java        # Jackson lazy field filter
│   │   │   │   │   └── dto/                          # Data Transfer Objects
│   │   │   │   │       ├── ClienteResponseDTO.java
│   │   │   │   │       ├── ProductoResponseDTO.java
│   │   │   │   │       ├── ProveedorDTO.java
│   │   │   │   │       └── ProveedorRegistroDTO.java
│   │   │   │   ├── presentation/                     # REST Controllers
│   │   │   │   │   ├── Login.java                    # Authentication endpoints
│   │   │   │   │   ├── Registro.java                 # Registration endpoint
│   │   │   │   │   ├── Proveedores.java              # Supplier management
│   │   │   │   │   ├── Clientes.java                 # Client CRUD
│   │   │   │   │   ├── Productos.java                # Product CRUD
│   │   │   │   │   ├── Facturar.java                 # Invoice creation
│   │   │   │   │   ├── Facturas.java                 # Invoice list & PDF
│   │   │   │   │   └── Perfil.java                   # Profile management
│   │   │   │   └── security/                         # Security layer
│   │   │   │       ├── UserDetailsImp.java           # UserDetails implementation
│   │   │   │       └── UserDetailsServiceImpl.java   # Custom UserDetailsService
│   │   │   └── resources/
│   │   │       ├── application.properties             # App configuration
│   │   │       └── static/                            # Frontend resources
│   │   │           ├── index.html                     # Entry point (redirects)
│   │   │           ├── css/
│   │   │           │   ├── menu.css                   # Navigation styles
│   │   │           │   └── style.css                  # Global styles
│   │   │           ├── js/
│   │   │           │   ├── login.js                   # Auth logic
│   │   │           │   └── menu.js                    # Navigation logic
│   │   │           ├── images/                        # Static assets
│   │   │           └── pages/                         # SPA pages
│   │   │               ├── bienvenida/                # Welcome page
│   │   │               │   ├── view.html
│   │   │               │   └── controller.js
│   │   │               ├── registro/                  # Registration page
│   │   │               │   ├── view.html
│   │   │               │   └── controller.js
│   │   │               ├── proveedores/               # Supplier management
│   │   │               │   ├── view.html
│   │   │               │   └── controller.js
│   │   │               ├── clientes/                  # Client management
│   │   │               │   ├── view.html
│   │   │               │   └── controller.js
│   │   │               ├── productos/                 # Product management
│   │   │               │   ├── view.html
│   │   │               │   └── controller.js
│   │   │               ├── facturar/                  # Invoice creation
│   │   │               │   ├── view.html
│   │   │               │   └── controller.js
│   │   │               ├── facturas/                  # Invoice list
│   │   │               │   ├── view.html
│   │   │               │   └── controller.js
│   │   │               └── perfil/                    # Profile page
│   │   │                   ├── view.html
│   │   │                   └── controller.js
│   │   └── test/
│   │       └── java/eif209/facturacioncsr/
│   │           └── FacturacionCsrApplicationTests.java
│   ├── pom.xml                                        # Maven dependencies
│   ├── mvnw                                           # Maven wrapper (Unix)
│   └── mvnw.cmd                                       # Maven wrapper (Windows)
└── README.md
```

---

## Getting Started

### Prerequisites

- **Java Development Kit (JDK)** 21 or higher
- **Apache Maven** 3.8+ (or use included Maven wrapper)
- **MySQL Server** 8.0+
- **Git** (for cloning the repository)

### Installation

1. Clone the repository:
   ```sh
   git clone https://github.com/isaacmendezr/springboot-invoice-fullstack.git
   cd springboot-invoice-fullstack
   ```

2. Navigate to the project directory:
   ```sh
   cd FacturacionCSR
   ```

### Database Setup

1. Start your MySQL server and create the database:
   ```sql
   CREATE DATABASE IF NOT EXISTS Facturacion;
   ```

2. Run the schema script:
   ```sh
   mysql -u root -p Facturacion < ../DB/Facturacion_Schema.sql
   ```

3. (Optional) Load sample data with pre-encrypted passwords:
   ```sh
   mysql -u root -p Facturacion < ../DB/Facturacion_Data.sql
   ```

   **Sample credentials after loading data:**
   - **Admin**: `admin` / `password`
   - **Active Supplier**: `proveedor1` / `password`
   - **Inactive Supplier**: `proveedor2` / `password`
   - **Pending Supplier**: `proveedor3` / `password`

### Configuration

Update the database credentials in `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/Facturacion
spring.datasource.username=root
spring.datasource.password=YourPassword
```

**Optional configurations:**
```properties
# Enable SQL logging
spring.jpa.show-sql=true

# Disable open-in-view (recommended for REST APIs)
spring.jpa.open-in-view=false

# Preserve column names as defined in @Column annotations
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

### Usage

1. Run the application using Maven:
   ```sh
   ./mvnw spring-boot:run
   ```

   Or on Windows:
   ```sh
   mvnw.cmd spring-boot:run
   ```

2. Access the application:
   ```
   http://localhost:8080
   ```

3. **Administrator workflow:**
   - Login with admin credentials at the welcome page
   - Navigate to "Proveedores" (Suppliers)
   - Review pending supplier registrations
   - Activate or deactivate supplier accounts

4. **Supplier workflow:**
   - Register a new account (initial status: PENDIENTE)
   - Wait for administrator activation
   - Login with activated credentials
   - Manage clients and products via respective sections
   - Create invoices by selecting client and adding products
   - View and export invoices to PDF

5. **API Testing:**
   ```sh
   # Test authentication
   curl -X POST http://localhost:8080/api/login/login \
     -H "Content-Type: application/json" \
     -d '{"nombreUsuario":"admin","contrasena":"password"}' \
     -c cookies.txt

   # Test authenticated endpoint
   curl -X GET http://localhost:8080/api/proveedores/listar \
     -b cookies.txt
   ```

---

## Security

The application implements comprehensive security features:

**Authentication:**
- Session-based authentication with HTTP-only cookies
- BCrypt password hashing with `{bcrypt}` prefix
- Custom `UserDetailsService` loading users from database
- Login/logout endpoints with proper session management

**Authorization:**
- Role-based access control (RBAC)
- **ADMINISTRADOR**: Full access to supplier management
- **PROVEEDOR**: Access to client, product, and invoice operations
- Method-level security with `@AuthenticationPrincipal`

**User States:**
- **PENDIENTE**: Newly registered users awaiting activation
- **ACTIVO**: Active users who can authenticate
- **INACTIVO**: Deactivated users (blocked from login)

**Security Configuration:**
- CSRF disabled for REST API compatibility
- Custom authentication entry point (HTTP 401)
- Lazy field filtering to prevent serialization issues
- Public endpoints: login, registration, static resources

---

## Frontend

The frontend is a Single Page Application (SPA) built with vanilla JavaScript:

**Architecture:**
- **HTML5**: Semantic markup with Bootstrap classes
- **CSS3**: Custom styles with responsive design
- **JavaScript (ES6+)**: Async/await with Fetch API
- **State Management**: Global `loginstate` object

**Key Features:**
- Dynamic menu rendering based on user role
- Form validation before API calls
- Error handling with user-friendly messages
- Modal dialogs for CRUD operations
- Real-time calculations for invoice totals
- Client-side routing with page controllers

**Page Structure:**
- Each page has its own `view.html` and `controller.js`
- Controllers handle API interactions and DOM manipulation
- Shared `login.js` and `menu.js` for authentication and navigation
- Modular design with separation of concerns

**API Communication:**
```javascript
// Example: Fetch with error handling
const response = await fetch('/api/productos', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(productData)
});

if (!response.ok) {
  errorMessage(response.status);
  return;
}
```

---

## License

Academic project developed for educational purposes.

## Contact

**Developer:** Isaac Méndez  
**Repository:** [springboot-invoice-fullstack](https://github.com/isaacmendezr/springboot-invoice-fullstack)  
**Previous Version:** [springboot-invoice-management](https://github.com/isaacmendezr/springboot-invoice-management)

---

**Note:** This project demonstrates modern fullstack development with Spring Boot REST API backend and Vanilla JavaScript SPA frontend, proper security implementation, and clean separation of concerns. It serves as an evolution from server-side rendering (Thymeleaf) to a decoupled fullstack architecture.
