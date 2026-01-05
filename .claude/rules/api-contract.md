# API Contract Documentation

This document defines the API contracts between the frontend and backend. **ALL API changes must be documented here BEFORE implementation.**

## Purpose

This file serves as the **single source of truth** for API contracts. It ensures:
- Frontend and backend stay in sync
- Breaking changes are identified early
- API versioning is tracked
- Documentation is always up-to-date

## Rules

1. **Document BEFORE implementing** - Write the contract first, then code
2. **Update both sides** - When changing an API, update frontend and backend together
3. **Version breaking changes** - Use `/api/v2/` for incompatible changes
4. **Include examples** - Show request/response examples
5. **Document errors** - List all possible error responses

## API Base URL

- **Development**: `http://localhost:8000`
- **Production**: Set via `VITE_API_BASE_URL` environment variable

## Authentication

All authenticated endpoints require a Bearer token in the Authorization header:

```
Authorization: Bearer <jwt_token>
```

### POST /api/auth/login

Authenticate user and receive JWT token.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123"
}
```

**Response (200):**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer"
}
```

**Errors:**
- `401 Unauthorized` - Invalid credentials
- `400 Bad Request` - Missing or invalid fields

---

### POST /api/auth/register

Register a new user account.

**Request:**
```json
{
  "email": "user@example.com",
  "username": "johndoe",
  "password": "SecurePass123"
}
```

**Response (201):**
```json
{
  "id": 1,
  "email": "user@example.com",
  "username": "johndoe",
  "is_active": true,
  "is_superuser": false,
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `409 Conflict` - Email or username already exists
- `400 Bad Request` - Invalid input (weak password, invalid email, etc.)

---

### GET /api/auth/me

Get current authenticated user information.

**Headers:**
```
Authorization: Bearer <token>
```

**Response (200):**
```json
{
  "id": 1,
  "email": "user@example.com",
  "username": "johndoe",
  "is_active": true,
  "is_superuser": false,
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `401 Unauthorized` - Invalid or expired token

---

## Items API

### GET /api/items

Get list of items with pagination and filtering.

**Query Parameters:**
- `skip` (integer, default: 0) - Number of items to skip
- `limit` (integer, default: 100, max: 100) - Number of items to return
- `category_id` (integer, optional) - Filter by category
- `is_available` (boolean, optional) - Filter by availability
- `min_price` (float, optional) - Minimum price filter
- `max_price` (float, optional) - Maximum price filter
- `search` (string, optional) - Search in name and description

**Response (200):**
```json
{
  "items": [
    {
      "id": 1,
      "name": "Laptop",
      "description": "High-performance laptop",
      "price": 999.99,
      "is_available": true,
      "quantity": 10,
      "owner_id": 1,
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 1,
  "skip": 0,
  "limit": 100
}
```

**Errors:**
- `400 Bad Request` - Invalid query parameters

---

### GET /api/items/{item_id}

Get a specific item by ID.

**Path Parameters:**
- `item_id` (integer, required) - Item ID

**Response (200):**
```json
{
  "id": 1,
  "name": "Laptop",
  "description": "High-performance laptop",
  "price": 999.99,
  "is_available": true,
  "quantity": 10,
  "owner_id": 1,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `404 Not Found` - Item does not exist

---

### POST /api/items

Create a new item.

**Headers:**
```
Authorization: Bearer <token>
```

**Request:**
```json
{
  "name": "Laptop",
  "description": "High-performance laptop",
  "price": 999.99,
  "is_available": true,
  "quantity": 10
}
```

**Response (201):**
```json
{
  "id": 1,
  "name": "Laptop",
  "description": "High-performance laptop",
  "price": 999.99,
  "is_available": true,
  "quantity": 10,
  "owner_id": 1,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `401 Unauthorized` - Not authenticated
- `400 Bad Request` - Invalid input (negatiprice, empty name, etc.)

---

### PUT /api/items/{item_id}

Update an existing item.

**Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
- `item_id` (integer, required) - Item ID

**Request (all fields optional):**
```json
{
  "name": "Updated Laptop",
  "description": "Updated description",
  "price": 899.99,
  "is_available": false,
  "quantity": 5
}
```

**Response (200):**
```json
{
  "id": 1,
  "name": "Updated Laptop",
  "description": "Updated description",
  "price": 899.99,
  "is_available": false,
  "quantity": 5,
  "owner_id": 1,
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-01-15T12:00:00Z"
}
```

**Errors:**
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Not the owner of the item
- `404 Not Found` - Item does not exist
- `400 Bad Request` - Invalid input

---

### DELETE /api/items/{item_id}

Delete an item.

**Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
- `item_id` (integer, required) - Item ID

**Response (204):**
No content

**Errors:**
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Not the owner of the item
- `404 Not Found` - Item does not exist

---

## Users API

### GET /api/users

Get list of users (admin only).

**Headers:**
```
Authorization: Bearer <token>
```

**Query Parameters:**
- `skip` (integer, default: 0) - Number of users to skip
- `limit` (integer, default: 100, max: 100) - Number of users to return

**Response (200):**
```json
{
  "users": [
    {
      "id": 1,
      "email": "user@example.com",
      "username": "johndoe",
      "is_active": true,
      "is_superuser": false,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ],
  "total": 1,
  "skip": 0,
  "limit": 100
}
```

**Errors:**
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Not a superuser

---

### GET /api/users/{user_id}

Get a specific user by ID.

**Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
- `user_id` (integer, required) - User ID

**Response (200):**
```json
{
  "id": 1,
  "email": "user@example.com",
  "username": "johndoe",
  "is_active": true,
  "is_superuser": false,
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Can only view own profile (unless superuser)
- `404 Not Found` - User does not exist

---

### PUT /api/users/{user_id}

Update user information.

**Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
- `user_id` (integer, required) - User ID

**Request (all fields optional):**
```json
{
  "email": "newemail@example.com",
  "username": "newusername",
  "password": "NewSecurePass123"
}
```

**Response (200):**
```json
{
  "id": 1,
  "email": "newemail@example.com",
  "username": "newusername",
  "is_active": true,
  "is_superuser": false,
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Can only update own profile (unless superuser)
- `404 Not Found` - User does not exist
- `409 Conflict` - Email or username already taken
- `400 Bad Request` - Invalid input

---

### DELETE /api/users/{user_id}

Delete a user account.

**Headers:**
```
Authorization: Bearer <token>
```

**Path Parameters:**
- `user_id` (integer, required) - User ID

**Response (204):**
No content

**Errors:**
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Can only d account (unless superuser)
- `404 Not Found` - User does not exist

---

## Categories API

### GET /api/categories

Get list of all categories.

**Response (200):**
```json
[
  {
    "id": 1,
    "name": "Electronics",
    "description": "Electronic devices and accessories",
    "created_at": "2024-01-15T10:30:00Z"
  },
  {
    "id": 2,
    "name": "Books",
    "description": "Books and publications",
    "created_at": "2024-01-15T10:30:00Z"
  }
]
```

---

### POST /api/categories

Create a new category (admin only).

**Headers:**
```
Authorization: Bearer <token>
```

**Request:**
```j
{
  "name": "Electronics",
  "description": "Electronic devices and accessories"
}
```

**Response (201):**
```json
{
  "id": 1,
  "name": "Electronics",
  "description": "Electronic devices and accessories",
  "created_at": "2024-01-15T10:30:00Z"
}
```

**Errors:**
- `401 Unauthorized` - Not authenticated
- `403 Forbidden` - Not a superuser
- `409 Conflict` - Category name already exists
- `400 Bad Request` - Invalid input

---

## Health Check

### GET /health

Check if the API is running.

**Response (200):**
```json
{
  "status":,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

## Error Response Format

All error responses follow this format:

```json
{
  "detail": "Error message describing what went wrong"
}
```

### Common HTTP Status Codes

- `200 OK` - Request succeeded
- `201 Created` - Resource created successfully
- `204 No Content` - Request succeeded, no content to return
- `400 Bad Request` - Invalid input or malformed request
- `401 Unauthorized` - Authentication required or failed
- `403 Forbidden` - Authenticated but not authorized
- `404 Not Found` - Resource does not exist
- `409 Conflict` - Resource conflict (duplicate, etc.)
- `422 Unprocessable Entitydation error
- `500 Internal Server Error` - Server error

---

## TypeScript Types (Frontend)

These types should be defined in `frontend/src/types/models.ts`:

```typescript
// User types
interface User {
  id: number;
  email: string;
  username: string;
  is_active: boolean;
  is_superuser: boolean;
  created_at: string;
}

interface UserCreate {
  email: string;
  username: string;
  password: string;
}

interface UserUpdate {
  email?: string;
  username?: string;
  password?: string;
}

// Item types
interface Item {
  id: number;
  name: string;
  description: string;
  price: number;
  is_available: boolean;
  quantity: number;
  owner_id: number;
  created_at: string;
  updated_at: string;
}

interface ItemCreate {
  name: string;
  description: string;
  price: number;
  is_available?: boolean;
  quantity?: number;
}

interface ItemUpdate {
  name?: string;
  description?: string;
  price?: number;
  is_available?: boolean;
  quantity?: number;
}

// Category types
interface Category {
  id: number;
  name: string;
  description: string;
  created_at: string;
}

interface CategoryCreate {
  name: string;
  description: string;
}

// Auth types
interface LoginRequest {
  email: string;
  password: string;
}

interface TokenResponse {
  access_token: string;
  token_type: string;
}

// Pagination types
interface PaginatedResponse<T> {
  items: T[];
  total: number;
  skip: number;
  limit: number;
}

// API Error type
interface ApiError {
  detail: string;
}
```

---

## Python Types (Backend)

These types should be defined in `backend/schemas.py`:

```python
from pydantic import BaseModel, EmailStr, Field
from typing import Optional, List
from datetime import datetime

# User schemas
class UserBase(BaseModel):
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)

class UserCreate(UserBase):
    password: str = Field(..., min_length=8, max_length=100)

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    username: Optional[str] = Field(None, min_length=3, max_length=50)
    password: Optional[str] = Field(None, min_length=8, max_length=100)

class UserResponse(UserBase):
    id: int
    is_active: bool
    is_superuser: bool
    created_at: datetime

    class Config:
        from_attributes = True

# Item schemas
class ItemBase(BaseModel):
    name: str = Field(..., min_length=1, max_length=100)
    description: str = Field(..., max_length=500)
    price: float = Field(..., gt=0)
    is_available: bool = True
    quantity: int = Field(default=0, ge=0)

class ItemCreate(ItemBase):
    pass

class ItemUpdate(BaseModel):
    name: Optional[str] = Field(None, min_length=1, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    price: Optional[float] = Field(None, gt=0)
    is_available: Optional[bool] = None
    quantity: Optional[int] = Field(None, ge=0)

class ItemResponse(ItemBase):
    id: int
    owner_id: int
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True

# Category schemas
class CategoryBase(BaseModel):
    name: str = Field(..., min_length=1, max_length=50)
    description: str = Field(..., max_length=200)

class CategoryCreate(CategoryBase):
    pass

class CategoryResponse(CategoryBase):
    id: int
    created_at: datetime

    class Config:
        from_attributes = True

# Auth schemas
class LoginRequest(BaseModel):
    email: EmailStr
    password: str

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"

# Pagination schema
class PaginatedResponse(BaseModel):
    items: List[Any]
    total: int
    skip: int
    limit: int
```

---

## Versioning Strategy

When making breaking changes:

1. **Create new version**: `/api/v2/items` instead of `/api/items`
2. **Maintain old version**: Keep v1 working for backward compatibility
3. **Document deprecation**: Add deprecation notice to old endpoints
4. **Set sunset date**: Give users time to migrate (e.g., 6 months)
5. **Update frontend**: Migrate to new version gradually

Example deprecation notice:

```python
@router.get("/api/items", deprecated=True)
async def get_items_v1():
    """
    DEPRECATED: Use /api/v2/items instead.
    This endpoint will be removed on 2024-07-01.
    """
    pass
```

---

## Testing API Contracts

### Backend Testing

```python
# tests/test_api_contract.py
def test_item_response_schema(client: TestClient):
    """Ensure item response matches contract"""
    response = client.get("/api/items/1")
    assert response.status_code == 200

    data = response.json()
    assert "id" in data
    assert "name" in data
    assert "price" in data
    assert isinstance(data["id"], int)
    assert isinstance(data["price"], float)
```

### Frontend Testing

```typescript
// tests/api-contract.test.ts
test('item response matches contract', async () => {
  const item = await fetchItem(1);

  expect(item).toHaveProperty('id');
  expect(item).toHaveProperty('name');
  expect(item).toHaveProperty('price');
  expect(typeof item.id).toBe('number');
  expect(typeof item.price).toBe('number');
});
```

---

## Change Log

Document all API changes here:

### 2024-01-15 - Initial API Design
- Created authentication endpoints
- Created items CRUD endpoints
- Created users CRUD endpoints
- Created categories endpoints

---

## Notes for Developers

1. **Always update this file first** before implementing API changes
2. **Test with actual HTTP requests** using FastAPI docs or Postman
3. **Keep types in sync** between frontend and backend
4. **Document all error cases** - don't surprise users
5. **Use semantic versioning** for breaking changes
6. **Validate on both sides** - frontend for UX, backend for security
