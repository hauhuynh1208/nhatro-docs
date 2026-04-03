Title: Seller creates buyer account
ID: US-4
Actor: seller
As a: seller
I want: create a buyer account by providing email, password, and basic info
So that: the new buyer can immediately log in and I can assign them to a house

Preconditions:

- Caller is authenticated as `seller` with a valid JWT
- Application is running

Trigger:

- Seller sends `POST /api/users` with `Authorization: Bearer {sellerToken}` and body `{ email, password, role: 3 }`

Role-based visibility:

- A seller can only see buyers that they created (`created_by = seller.id`)
- Sellers cannot see buyers created by other sellers
- `admin` can see all buyers regardless of who created them

Acceptance Criteria (AC):

1. Request body is validated — `email` must be a valid email format, `password` must be at least 8 characters, `role` must be `3` (buyer); missing or invalid fields return HTTP 400.
2. On success, a user record is created with `role=3`, `isActive=true`, `created_by=seller.id`, and a bcrypt-hashed password; HTTP 201 is returned with `{ id, email, role, isActive, createdAt }` — password is never returned.
3. If the email already exists, the API returns HTTP 409 and no duplicate record is created.
4. Requests without a valid JWT return HTTP 401.
5. A `seller` attempting to create `role=1` (admin) or `role=2` (seller) returns HTTP 403.
6. The newly created buyer can immediately authenticate via `POST /api/auth/login` using the credentials provided.
7. When `GET /api/users?role=3` is called by the seller, only buyers with `created_by = seller.id` are returned — buyers created by other sellers are not visible.
8. When `GET /api/users?role=3` is called by a different seller, the buyer created in step 2 does not appear in the results.

Steps for AI agent (ordered):

1. POST `/api/auth/login` with seller A credentials — capture `sellerAToken`.
2. POST `/api/users` with `Authorization: Bearer {sellerAToken}` and body `{"email":"buyerA@test.com","password":"Test@1234","role":3}` — expect 201 and `{ id, email, role: 3, isActive: true, createdAt }`, no password field.
3. POST `/api/auth/login` with the new buyer credentials `{"email":"buyerA@test.com","password":"Test@1234"}` — expect 200 and `accessToken`.
4. GET `/api/users?role=3` with `Authorization: Bearer {sellerAToken}` — expect 200 and list containing `buyerA@test.com`.
5. POST `/api/auth/login` with seller B credentials — capture `sellerBToken`.
6. GET `/api/users?role=3` with `Authorization: Bearer {sellerBToken}` — expect 200 and list NOT containing `buyerA@test.com`.
7. POST `/api/users` with `Authorization: Bearer {sellerAToken}` and body `{"email":"buyerA@test.com","password":"Test@1234","role":3}` (duplicate) — expect 409.
8. POST `/api/users` with `Authorization: Bearer {sellerAToken}` and body `{"role":2,...}` — expect 403.
9. POST `/api/users` with `Authorization: Bearer {sellerAToken}` and body `{"role":1,...}` — expect 403.
10. Return structured result: `{"status":"ok|failed","evidence":{...}}`

Machine-readable JSON:

```json
{
  "id": "US-4",
  "title": "Seller creates buyer account",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "create a buyer account with email, password, and role",
  "so_that": "new buyer can immediately log in and be assigned to a house",
  "preconditions": [
    "seller is authenticated with valid JWT",
    "application is running"
  ],
  "trigger": "POST /api/users",
  "role_permissions": {
    "seller_can_create": [3],
    "seller_cannot_create": [1, 2]
  },
  "visibility_rule": "seller can only see buyers with created_by = seller.id",
  "acceptance_criteria": [
    "400 on invalid or missing email/password/role",
    "201 with user payload (no password) on success, role=3, isActive=true, created_by=seller.id",
    "409 on duplicate email",
    "401 without valid JWT",
    "403 when seller tries to create role 1 (admin) or role 2 (seller)",
    "new buyer can log in immediately with provided credentials",
    "GET /api/users?role=3 returns only buyers created by the calling seller",
    "buyers created by other sellers are not visible to this seller"
  ],
  "steps": [
    {
      "action": "seller A login",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "sellerA@test.com", "password": "Test@1234" }
      },
      "expect": "200 and accessToken"
    },
    {
      "action": "seller A creates buyer",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {sellerAToken}" },
        "body": {
          "email": "buyerA@test.com",
          "password": "Test@1234",
          "role": 3
        }
      },
      "expect": "201 and { id, email, role: 3, isActive: true, createdAt } — no password field"
    },
    {
      "action": "new buyer logs in",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "buyerA@test.com", "password": "Test@1234" }
      },
      "expect": "200 and accessToken with role: 3"
    },
    {
      "action": "seller A lists own buyers",
      "api": {
        "method": "GET",
        "path": "/api/users?role=3",
        "headers": { "Authorization": "Bearer {sellerAToken}" }
      },
      "expect": "200 and list containing buyerA@test.com"
    },
    {
      "action": "seller B login",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "sellerB@test.com", "password": "Test@1234" }
      },
      "expect": "200 and accessToken"
    },
    {
      "action": "seller B lists own buyers — must NOT see seller A's buyer",
      "api": {
        "method": "GET",
        "path": "/api/users?role=3",
        "headers": { "Authorization": "Bearer {sellerBToken}" }
      },
      "expect": "200 and list NOT containing buyerA@test.com"
    },
    {
      "action": "duplicate email",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {sellerAToken}" },
        "body": {
          "email": "buyerA@test.com",
          "password": "Test@1234",
          "role": 3
        }
      },
      "expect": "409"
    },
    {
      "action": "seller tries to create seller (forbidden)",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {sellerAToken}" },
        "body": {
          "email": "seller2@test.com",
          "password": "Test@1234",
          "role": 2
        }
      },
      "expect": "403"
    },
    {
      "action": "seller tries to create admin (forbidden)",
      "api": {
        "method": "POST",
        "path": "/api/users",
        "headers": { "Authorization": "Bearer {sellerAToken}" },
        "body": {
          "email": "admin2@test.com",
          "password": "Test@1234",
          "role": 1
        }
      },
      "expect": "403"
    }
  ],
  "data": {
    "user": {
      "id": "uuid",
      "email": "string",
      "role": 3,
      "isActive": true,
      "created_by": "uuid (seller.id)",
      "createdAt": "timestamp"
    }
  },
  "mocks": {
    "POST /api/auth/login (sellerA)": {
      "status": 200,
      "body": {
        "accessToken": "{sellerAToken}",
        "user": { "id": "seller-a-id", "role": 2 }
      }
    },
    "POST /api/users (create buyer)": {
      "status": 201,
      "body": {
        "id": "buyer-a-id",
        "email": "buyerA@test.com",
        "role": 3,
        "isActive": true,
        "createdAt": "2026-04-03T00:00:00Z"
      }
    },
    "GET /api/users?role=3 (sellerA)": {
      "status": 200,
      "body": [{ "id": "buyer-a-id", "email": "buyerA@test.com", "role": 3 }]
    },
    "GET /api/users?role=3 (sellerB)": {
      "status": 200,
      "body": []
    }
  }
}
```

Notes:

- `created_by` field on the `users` table is set to the calling seller's `id` at creation time — this is the primary key for scoping visibility.
- The `GET /api/users?role=3` endpoint must filter by `created_by = current_user.id` when called by a `seller`. No additional query param is needed from the client; the server infers the scope from the JWT.
- `admin` bypasses the `created_by` filter and can see all buyers.
- Password is hashed with bcrypt and never returned in any response.
