# API Specification

> Define your API endpoints here, or paste your OpenAPI/Swagger spec.
> Agent will use this to ensure backend implementation matches contract.

---

## Base URL

```
Development: http://localhost:3000/api
Production:  https://api.yourdomain.com/api
```

## Authentication

**Method:** Bearer Token (JWT)

```
Authorization: Bearer <token>
```

## Response Format

All responses follow this shared contract:

```typescript
// Success
{ "success": true, "data": <payload> }

// Error
{ "success": false, "error": "<message>", "details": [...] }
```

---

## Endpoints

### Auth

#### POST /auth/register
Register a new user.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "min8chars",
  "name": "John Doe"
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "token": "jwt_token",
    "user": { "id": "...", "email": "...", "name": "..." }
  }
}
```

**Errors:** 400 (validation), 409 (email exists)

---

#### POST /auth/login
Authenticate user.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "token": "jwt_token",
    "user": { "id": "...", "email": "...", "name": "..." }
  }
}
```

**Errors:** 400 (validation), 401 (invalid credentials)

---

### [Resource Name] (e.g. Users)

#### GET /users
Get list of users. Requires auth.

**Query params:**
- `page` (number, default 1)
- `limit` (number, default 20)
- `search` (string, optional)

**Response 200:**
```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 100,
    "page": 1,
    "limit": 20
  }
}
```

---

#### GET /users/:id
Get single user. Requires auth.

**Response 200:**
```json
{
  "success": true,
  "data": { "id": "...", "email": "...", "name": "..." }
}
```

**Errors:** 404 (not found)

---

#### POST /users
Create user. Requires auth + admin role.

**Request:** `{ "email": "...", "name": "...", "role": "USER" }`

**Response 201:** `{ "success": true, "data": { ...user } }`

---

#### PUT /users/:id
Update user. Requires auth.

**Request:** `{ "name": "..." }` (partial update)

**Response 200:** `{ "success": true, "data": { ...updated_user } }`

---

#### DELETE /users/:id
Delete user. Requires auth + admin role.

**Response 200:** `{ "success": true, "data": null }`

---

## Error Codes

| Code | Meaning |
|------|---------|
| 400 | Bad Request — validation failed |
| 401 | Unauthorized — missing or invalid token |
| 403 | Forbidden — insufficient permissions |
| 404 | Not Found |
| 409 | Conflict — duplicate resource |
| 429 | Too Many Requests — rate limited |
| 500 | Internal Server Error |

---

## Rate Limiting

- **Auth endpoints:** 10 requests / minute per IP
- **API endpoints:** 100 requests / minute per user

---

## Webhooks (if applicable)

| Event | Payload |
|-------|---------|
| `user.created` | `{ userId, email, createdAt }` |
| `order.completed` | `{ orderId, total, userId }` |
