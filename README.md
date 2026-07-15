# Microservices User Management & Role-Based Access Control (RBAC) System

A robust, enterprise-grade authentication and user management system designed using a **Microservices Architecture**. This solution features fine-grained Role-Based Access Control (RBAC) maps, stateless JWT session tokens, an API Gateway, and database separation.

---

## 🏛️ System Architecture

The application is split into four decoupled services, adhering to the **Database-per-Service** design pattern:

1. **API Gateway (Port 5000)**: Serves as the single access point, handling CORS and forwarding requests to the appropriate service.
2. **Auth Service (Port 5001)**: Manages credential storage, password hashing (bcrypt), login validation, and JWT generation. Uses `auth.db`.
3. **User & RBAC Service (Port 5002)**: Handles user profiles, roles, permissions, and mappings. Performs compensating transactions to rollback failures on microservice API breaks. Uses `user_rbac.db`.
4. **Vite React Frontend (Port 3001)**: Responsive SPA styled with TailwindCSS, containing dashboard views, route guards, and permission-based rendering.

```
React Client (3001) ──> API Gateway (5000)
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

## ⚡ Quick Start: Running the App

### Option A: Running with Docker Compose (Recommended)
Make sure you have Docker installed. Spin up the entire multi-service stack in one command:
```bash
docker-compose up --build
```
Once booted, visit:
* Frontend Console: **`http://localhost:3000`** (or **`http://localhost:3001`** if port 3000 is occupied)
* API Gateway Health: **`http://localhost:5000/health`**

### Option B: Running Locally (Without Docker)
Ensure you have **Node.js (v18+)** installed. In the `rbac-system/` root directory:

1. **Install all service dependencies**:
   ```bash
   npm run install:all
   ```
2. **Start all services concurrently**:
   ```bash
   npm run dev
   ```
3. Visit the dashboard at **`http://localhost:3001`** (or **`http://localhost:3000`**).

---

## 🔑 Default Credentials

The databases automatically seed a default administrator account on first startup:
* **Email**: `admin@rbac.com`
* **Password**: `admin123`

---

## 📖 API Documentation

All routes route through the **API Gateway** on Port `5000`.

### 1. Authentication Service (`/api/auth`)
* **`POST /api/auth/login`**: Authenticates credentials and returns a JWT token.
  * *Request Body*: `{ "email": "admin@rbac.com", "password": "admin123" }`
  * *Response*: `{ "token": "JWT_TOKEN", "user": { "id": 1, "email": "admin@rbac.com" } }`

### 2. User & RBAC Service (`/api/users`, `/api/roles` & `/api/permissions`)
*All routes below require authentication by passing `Authorization: Bearer <JWT_TOKEN>`.*

* **`GET /api/users/me/permissions`**: Retrieves the active profile, role name, and authorized permissions.
* **`GET /api/users`**: List user profiles (Supports query pagination: `GET /api/users?page=1&limit=5`).
* **`POST /api/users`**: Creates a user profile and registers credentials in the Auth Service (Requires `users:create` permission).
  * *Request Body*: `{ "first_name": "John", "last_name": "Doe", "email": "john@rbac.com", "password": "password123", "role": "User" }`
* **`DELETE /api/users/:id`**: Removes user profile and credentials in the Auth Service (Requires `users:delete` permission).
* **`GET /api/roles`**: List roles and their permission mappings (Requires `roles:read` permission).
* **`POST /api/roles/assign`**: Re-assigns a user to a different role (Requires `roles:assign` permission).
  * *Request Body*: `{ "userId": 2, "roleName": "Admin" }`
* **`POST /api/roles/permissions`**: Re-maps permissions to a specific role ID (Requires `roles:assign` permission).
  * *Request Body*: `{ "roleId": 2, "permissionIds": [2, 3] }`
* **`GET /api/permissions`**: Lists all defined system permission nodes.

---

## 🛡️ Assumptions & Design Decisions
* **Database Migrations Reference**: The raw SQL schema statements used to build the tables are documented inside the [database/](file:///c:/Users/KrishnadasB/Desktop/Edaddy/rbac-system/database/) directory for production migration references.
* **Auto-Schema Setup**: For reviewer convenience, the SQLite databases are auto-created and seeded on startup. This allows reviewers to evaluate the project out-of-the-box with zero database configuration.
* **Compensating Transactions**: In distributed systems, user registration requires creating data in the User database first, followed by credentials in the Auth database. If the credentials generation step fails, the system automatically triggers a rollback to purge user-profiles, keeping databases in sync.
* **Self-Deletion & lockout safeguards**: Default admin account deletion or modification is blocked at both the API and database levels.
* **Granular Sidebar Navigation**: Split the UI dashboard sidebar into modular sections:
  * *Users*: Features paginated profiles table and a floating User Creation Modal.
  * *Roles & Access*: Focuses strictly on mapping permissions checkboxes to role definitions.
