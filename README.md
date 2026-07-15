# Microservices User Management & Role-Based Access Control (RBAC) System

A robust, enterprise-grade authentication and user management system designed using a **Microservices Architecture**. This solution features fine-grained Role-Based Access Control (RBAC) maps, stateless JWT session tokens, an API Gateway, and database separation.

---

## 🏛️ System Architecture

The application is split into four decoupled services, adhering to the **Database-per-Service** design pattern:

1. **API Gateway (Port 5000)**: Serves as the single access point, handling CORS and forwarding requests to the appropriate service.
2. **Auth Service (Port 5001)**: Manages credential storage, password hashing (bcrypt), login validation, and JWT generation. Uses `auth.db`.
3. **User & RBAC Service (Port 5002)**: Handles user profiles, roles, permissions, and mappings. Performs compensating transactions to rollback failures on microservice API breaks. Uses `user_rbac.db`.
4. **Vite React Frontend (Port 3000)**: Responsive SPA styled with TailwindCSS, containing dashboard views, route guards, and permission-based rendering.

```
React Client (3000) ──> API Gateway (5000)
                            ├──> Auth Service (5001) ──> auth.db
                            └──> User & RBAC Service (5002) ──> user_rbac.db
```

---

## 📂 Backend Code Architecture (4-Layer Clean Architecture)

Both microservices are built using a production-grade **4-Layer Clean Architecture** pattern to guarantee maintainability and separation of concerns:

```
Route (src/routes/) ──> Controller (src/controllers/) ──> Service (src/services/) ──> Repository (src/repositories/) ──> SQL Queries (src/queries/)
```

* **Route Layer**: Directs incoming requests to controllers.
* **Controller Layer**: Extracts request payloads, handles HTTP response codes, and delegates to services.
* **Service Layer**: Implements business rules, coordinates network API calls, and performs transaction rollbacks.
* **Repository Layer**: Directly interfaces with database drivers.
* **Queries File**: Completely isolates raw SQL query strings, making it trivial to swap databases in the future (e.g. from SQLite to PostgreSQL).

---

## 📥 Clone the Repository

This project uses **Git submodules** to organize the microservices. Clone the repository with all submodules using:

```bash
git clone --recurse-submodules https://github.com/Krishnadaskrish/access-control-platform.git
cd access-control-platform
```

If you have already cloned the repository without submodules, initialize and fetch them using:

```bash
git submodule update --init --recursive
```

After cloning, the project structure will be:

```text
access-control-platform/
│
├── auth-service/
├── user-service/
├── gateway/
├── frontend/
├── docker-compose.yml
└── README.md
```

---

## ⚡ Quick Start: Running the App

### Option A: Running Locally (Recommended)
Ensure you have **Node.js (v18+)** installed. In the `access-control-platform/` root directory:

1. **Install all service dependencies**:
   ```bash
   npm run install:all
   ```
2. **Start all services concurrently**:
   ```bash
   npm run dev
   ```
3. Visit the dashboard at **`http://localhost:3000`** (or **`http://localhost:3001`**).

### Option B: Running with Docker Compose (with docker)
Make sure you have Docker installed. Spin up the entire multi-service stack in one command:
```bash
docker compose up --build
```
Once booted, visit:
* Frontend Console: **`http://localhost:3000`** (or **`http://localhost:3001`** if port 3000 is occupied)
* API Gateway Health: **`http://localhost:5000/health`**

---

## 🔑 Default Credentials

The databases automatically seed a default administrator account on first startup:
* **Email**: `admin@gmail.com`
* **Password**: `admin123`

---

## 📖 API Documentation

All routes route through the **API Gateway** on Port `5000`.

### 1. Authentication Service (`/api/auth`)
* **`POST /api/auth/login`**: Authenticates credentials and returns a JWT token.
  * *Request Body*: `{ "email": "admin@gmail.com", "password": "admin123" }`
  * *Response*: `{ "token": "JWT_TOKEN", "user": { "id": 1, "email": "admin@gmail.com" } }`

### 2. User & RBAC Service (`/api/users`, `/api/roles` & `/api/permissions`)
*All routes below require authentication by passing `Authorization: Bearer <JWT_TOKEN>`.*

* **`GET /api/users/me/permissions`**: Retrieves the active profile, role name, and authorized permissions.
* **`GET /api/users`**: List user profiles (Supports query pagination: `GET /api/users?page=1&limit=5`).
* **`POST /api/users`**: Creates a user profile and registers credentials in the Auth Service (Requires `users:create` permission).
  * *Request Body*: `{ "first_name": "Krishnadas", "last_name": "B", "email": "[EMAIL_ADDRESS]", "password": "password123", "role": "User" }`
* **`DELETE /api/users/:id`**: Removes user profile and credentials in the Auth Service (Requires `users:delete` permission).
* **`GET /api/roles`**: List roles and their permission mappings (Requires `roles:read` permission).
* **`POST /api/roles/assign`**: Re-assigns a user to a different role (Requires `roles:assign` permission).
  * *Request Body*: `{ "userId": 2, "roleName": "Admin" }`
* **`POST /api/roles/permissions`**: Re-maps permissions to a specific role ID (Requires `roles:assign` permission).
  * *Request Body*: `{ "roleId": 2, "permissionIds": [2, 3] }`
* **`GET /api/permissions`**: Lists all defined system permission nodes.

---

## 🛡️ Assumptions & Design Decisions
* **Database Migrations Reference**: The raw SQL schema statements used to build the tables are documented inside the [database/](./database/) directory for production migration references.
* **Auto-Schema Setup**: No manual database configuration is required. The application uses SQLite, and the databases are automatically created, the schema is initialized, and the default seed data is inserted during the first startup. This allows reviewers to run the project out of the box without any additional database setup.
* **Compensating Transactions**: In distributed systems, user registration requires creating data in the User database first, followed by credentials in the Auth database. If the credentials generation step fails, the system automatically triggers a rollback to purge user-profiles, keeping databases in sync.
* **Admin Protection**: The default administrator account cannot be deleted or modified, preventing accidental lockout.
* **Sidebar Navigation**: Split the UI dashboard sidebar into modular sections:
  * *Users*: Features paginated profiles table and a floating User Creation Modal.
  * *Roles & Access*: Focuses strictly on mapping permissions checkboxes to role definitions.
