Title: Create user account
ID: US-3
Actor: admin, seller
As a: admin or seller
I want: create a user account by providing an email, password, and role
So that: the new user can immediately log in with the credentials I provided

Preconditions:

- Caller is authenticated with a valid JWT
- Application is running

Trigger:

- Caller sends `POST /api/users` with a Bearer token and user credentials

Role-based permissions:

| Caller role | Value | Can create roles          |
| ----------- | ----- | ------------------------- |
| `admin`     | `1`   | `2` (seller), `3` (buyer) |
| `seller`    | `2`   | `3` (buyer) only          |
| `buyer`     | `3`   | _(not allowed)_           |
| _(anyone)_  | —     | `1` (admin) — never       |

Acceptance Criteria (AC):

1. Request body is validated — `email` must be a valid email format, `password` must be at least 8 characters, `role` must be `2` (seller) or `3` (buyer); missing or invalid fields return HTTP 400.
2. On success, a user record is created with the requested `role`, `isActive=true`, and a bcrypt-hashed password; HTTP 201 is returned with `{ id, email, role, isActive, createdAt }` (password is never returned).
3. If the email already exists, the API returns HTTP 409 and no duplicate record is created.
4. Requests without a valid JWT return HTTP 401.
5. A `seller` caller attempting to create role `2` (seller) returns HTTP 403. A `buyer` caller returns HTTP 403 for any role.
6. Attempting to create role `1` (admin) always returns HTTP 403, regardless of caller role.
7. The newly created user can immediately authenticate via `POST /api/auth/login` using the credentials provided by the caller.

Steps for AI agent (ordered):

1. POST `/api/auth/login` with admin credentials — capture `adminToken`.
2. POST `/api/users` with `Authorization: Bearer {adminToken}` and body `{"email":"seller@test.com","password":"Test@1234","role":2}` — expect 201 and seller payload.
3. POST `/api/auth/login` with the new seller credentials — expect 200 and `role: 2`.
4. POST `/api/auth/login` with seller credentials — capture `sellerToken`.
5. POST `/api/users` with `Authorization: Bearer {sellerToken}` and body `{"email":"buyer@test.com","password":"Test@1234","role":3}` — expect 201 and buyer payload.
6. POST `/api/users` with `Authorization: Bearer {sellerToken}` and body `{"role":2,...}` — expect 403.
7. POST `/api/users` with `Authorization: Bearer {adminToken}` and body `{"role":1,...}` — expect 403.
8. POST `/api/users` with the same email as step 2 — expect 409.
9. POST `/api/users` without Authorization header — expect 401.
10. Return structured result: `{"status":"ok","evidence":{...}}`

Machine-readable JSON:

```json
{
  "id": "US-3",
  "title": "Create user account",
  "actor": "admin or seller",
  "as_a": "admin or seller",
  "i_want": "create a user account with email, password, and role",
  "so_that": "new user can immediately log in",
  "preconditions": ["caller authenticated", "application running"],
  "trigger": "POST /api/users",
  "role_permissions": {
    "1_admin": [2, 3],
    "2_seller": [3],
    "3_buyer": [],
    "forbidden_for_all": [1]
  },
  "acceptance_criteria": [
    "400 on invalid or missing email/password/role",
    "role must be 2 (seller) or 3 (buyer) — 400 otherwise",
    "201 with user payload (no password) on success",
    "409 on duplicate email",
    "401 without valid JWT",
    "403 when caller lacks permission for the requested role",
    "403 when trying to create an admin account",
    "new user can log in immediately with provided credentials"
  ],
  "steps": [
    {
      "action": "admin login",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "admin@nhatro.com", "password": "Admin@123" }
      },
      "expect": "200 and accessToken"
    },
    {
      "action": "admin creates seller",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {adminToken}" },
        "body": {
          "email": "seller@test.com",
          "password": "Test@1234",
          "role": 2
        }
      },
      "expect": "201 and { id, email, role: 2, isActive: true, createdAt }"
    },
    {
      "action": "seller login",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "seller@test.com", "password": "Test@1234" }
      },
      "expect": "200 and role: 2"
    },
    {
      "action": "seller creates buyer",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {sellerToken}" },
        "body": {
          "email": "buyer@test.com",
          "password": "Test@1234",
          "role": 3
        }
      },
      "expect": "201 and { id, email, role: 3, isActive: true, createdAt }"
    },
    {
      "action": "seller tries to create seller (forbidden)",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {sellerToken}" },
        "body": {
          "email": "seller2@test.com",
          "password": "Test@1234",
          "role": 2
        }
      },
      "expect": "403 forbidden"
    },
    {
      "action": "admin tries to create admin (forbidden)",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {adminToken}" },
        "body": {
          "email": "admin2@test.com",
          "password": "Test@1234",
          "role": 1
        }
      },
      "expect": "403 forbidden"
    },
    {
      "action": "duplicate email",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {adminToken}" },
        "body": {
          "email": "seller@test.com",
          "password": "Test@1234",
          "role": 2
        }
      },
      "expect": "409 conflict"
    },
    {
      "action": "no token",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "body": {
          "email": "other@test.com",
          "password": "Test@1234",
          "role": 3
        }
      },
      "expect": "401 unauthorized"
    }
  ]
}
```

Notes:

- Credentials are set by the caller and communicated to the new user out-of-band.
- No email invitation or onboarding flow — accounts are active immediately.
- Password policy: minimum 8 characters (enforced by API validation).
- `role` values: `1` = admin, `2` = seller, `3` = buyer. Role `1` (admin) cannot be created via this API; admin accounts are provisioned via seed script only.
