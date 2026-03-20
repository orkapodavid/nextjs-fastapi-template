# Backend API Reference

This page documents all API endpoints exposed by the FastAPI backend, including authentication routes (provided by `fastapi-users`), user management routes, and custom item CRUD routes.

All endpoints return JSON. Authentication is via JWT Bearer tokens unless noted otherwise.

---

## Authentication

These endpoints are provided by `fastapi-users` and mounted under the `/auth` prefix.

### Login

```
POST /auth/jwt/login
```

Obtain a JWT access token using email and password credentials.

**Request body** (form data):

| Field | Type | Required | Description |
|---|---|---|---|
| `username` | `string` | Yes | User's email address |
| `password` | `string` | Yes | User's password |

**Response** `200`:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer"
}
```

**Errors**: `400` Bad credentials · `422` Validation error

---

### Logout

```
POST /auth/jwt/logout
```

Invalidate the current token. Requires authentication.

**Headers**: `Authorization: Bearer <token>`

**Response** `200`: `null`

---

### Register

```
POST /auth/register
```

Create a new user account.

**Request body** (JSON):

```json
{
  "email": "user@example.com",
  "password": "SecurePass1!"
}
```

!!! note "Password Requirements"
    - Minimum 8 characters
    - At least one uppercase letter
    - At least one special character (`!@#$%^&*(),.?":{}|<>`)
    - Must not contain the user's email address

**Response** `201`:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "is_active": true,
  "is_superuser": false,
  "is_verified": false
}
```

**Errors**: `400` User already exists or invalid password · `422` Validation error

---

### Forgot Password

```
POST /auth/forgot-password
```

Request a password reset email. Always returns `202` regardless of whether the email exists (to prevent user enumeration).

**Request body** (JSON):

```json
{
  "email": "user@example.com"
}
```

**Response** `202`: `null`

---

### Reset Password

```
POST /auth/reset-password
```

Confirm a password reset using the token received via email.

**Request body** (JSON):

```json
{
  "token": "reset-token-from-email",
  "password": "NewSecurePass1!"
}
```

**Response** `200`: `null`

**Errors**: `400` Invalid/expired token or invalid password · `422` Validation error

---

### Verify Email

```
POST /auth/verify
```

Verify a user's email address using a verification token.

**Request body** (JSON):

```json
{
  "token": "verification-token"
}
```

**Response** `200`:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "is_active": true,
  "is_superuser": false,
  "is_verified": true
}
```

**Errors**: `400` Invalid token or already verified · `422` Validation error

---

### Request Verify Token

```
POST /auth/request-verify-token
```

Request a new email verification token.

**Request body** (JSON):

```json
{
  "email": "user@example.com"
}
```

**Response** `202`: `null`

---

## Users

User management endpoints, mounted under `/users`.

### Get Current User

```
GET /users/me
```

Retrieve the currently authenticated user's profile.

**Headers**: `Authorization: Bearer <token>`

**Response** `200`:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "user@example.com",
  "is_active": true,
  "is_superuser": false,
  "is_verified": true
}
```

**Errors**: `401` Not authenticated

---

### Update Current User

```
PATCH /users/me
```

Update the currently authenticated user's profile. All fields are optional.

**Headers**: `Authorization: Bearer <token>`

**Request body** (JSON):

```json
{
  "email": "new-email@example.com",
  "password": "NewPassword1!"
}
```

**Response** `200`: Updated user object

**Errors**: `401` Not authenticated · `400` Invalid update · `422` Validation error

---

### Get User by ID <small>(superuser only)</small>

```
GET /users/{id}
```

Retrieve any user's profile by UUID. Requires superuser privileges.

**Headers**: `Authorization: Bearer <token>`

**Response** `200`: User object

**Errors**: `401` Not authenticated · `403` Not a superuser · `404` User not found

---

### Update User <small>(superuser only)</small>

```
PATCH /users/{id}
```

Update any user's profile by UUID. Requires superuser privileges.

**Headers**: `Authorization: Bearer <token>`

**Response** `200`: Updated user object

**Errors**: `401` Not authenticated · `403` Not a superuser · `404` User not found

---

### Delete User <small>(superuser only)</small>

```
DELETE /users/{id}
```

Delete a user by UUID. Requires superuser privileges. Cascades to delete all associated items.

**Headers**: `Authorization: Bearer <token>`

**Response** `204`: No content

**Errors**: `401` Not authenticated · `403` Not a superuser · `404` User not found

---

## Items

Custom CRUD endpoints for managing items. All endpoints require authentication and scope items to the authenticated user. Mounted under `/items`.

### List Items

```
GET /items/?page=1&size=10
```

Retrieve a paginated list of items belonging to the authenticated user.

**Headers**: `Authorization: Bearer <token>`

**Query parameters**:

| Parameter | Type | Default | Constraints | Description |
|---|---|---|---|---|
| `page` | `int` | `1` | `≥ 1` | Page number |
| `size` | `int` | `10` | `1–100` | Items per page |

**Response** `200`:

```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Example Item",
      "description": "An item description",
      "quantity": 5,
      "user_id": "660e8400-e29b-41d4-a716-446655440000"
    }
  ],
  "total": 42,
  "page": 1,
  "size": 10,
  "pages": 5
}
```

**Errors**: `401` Not authenticated · `422` Validation error

---

### Create Item

```
POST /items/
```

Create a new item for the authenticated user.

**Headers**: `Authorization: Bearer <token>`

**Request body** (JSON):

```json
{
  "name": "New Item",
  "description": "Optional description",
  "quantity": 3
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `name` | `string` | Yes | Item name |
| `description` | `string` | No | Item description |
| `quantity` | `integer` | No | Item quantity |

**Response** `200`:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "New Item",
  "description": "Optional description",
  "quantity": 3,
  "user_id": "660e8400-e29b-41d4-a716-446655440000"
}
```

**Errors**: `401` Not authenticated · `422` Validation error

---

### Delete Item

```
DELETE /items/{item_id}
```

Delete an item by UUID. Only the item's owner can delete it.

**Headers**: `Authorization: Bearer <token>`

**Path parameters**:

| Parameter | Type | Description |
|---|---|---|
| `item_id` | `UUID` | The item's unique identifier |

**Response** `200`:

```json
{
  "message": "Item successfully deleted"
}
```

**Errors**: `401` Not authenticated · `404` Item not found or not authorised · `422` Validation error

---

## Schemas

### User Schemas

| Schema | Fields | Usage |
|---|---|---|
| `UserRead` | `id`, `email`, `is_active`, `is_superuser`, `is_verified` | All user response bodies |
| `UserCreate` | `email`, `password` | Registration request body |
| `UserUpdate` | `email?`, `password?` | User update request body |

### Item Schemas

| Schema | Fields | Usage |
|---|---|---|
| `ItemBase` | `name`, `description?`, `quantity?` | Base fields (internal) |
| `ItemCreate` | Inherits `ItemBase` | Create item request body |
| `ItemRead` | Inherits `ItemBase` + `id`, `user_id` | Item response body |

---

## Interactive Docs

When running the development server, interactive API documentation is available at:

- **Swagger UI**: [http://localhost:8000/docs](http://localhost:8000/docs)
- **ReDoc**: [http://localhost:8000/redoc](http://localhost:8000/redoc)
