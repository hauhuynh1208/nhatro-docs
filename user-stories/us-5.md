Title: Seller creates a price
ID: US-5
Actor: seller
As a: seller
I want: create a price by providing a name and a unit price
So that: I can later assign the price to a house or a group so it is used when calculating the monthly bill

Preconditions:

- Caller is authenticated as `seller` with a valid JWT
- Application is running

Trigger:

- Seller sends `POST /api/prices` with `Authorization: Bearer {sellerToken}` and request body

Required fields:

| Field        | Type    | Description                                       | Example     |
| ------------ | ------- | ------------------------------------------------- | ----------- |
| `name`       | string  | Display name for the price (chosen by the seller) | "tien dien" |
| `unit_price` | decimal | Price per unit; default currency is VND           | 2500        |

Acceptance Criteria (AC):

1. `name` and `unit_price` are required; missing or invalid fields return HTTP 400.
2. `unit_price` must be a positive number; values ≤ 0 return HTTP 400.
3. On success, a price record is created with `seller_id = caller.id`; HTTP 201 is returned with `{ id, name, unit_price, seller_id, createdAt }`.
4. Requests without a valid JWT return HTTP 401.
5. A `buyer` calling this endpoint returns HTTP 403.
6. `GET /api/prices` returns only prices belonging to the calling seller (`seller_id = caller.id`) — prices created by other sellers are not visible.
7. Two prices belonging to the same seller may share the same name (no uniqueness constraint on name).
8. A newly created price has no house or group assignment yet — those are separate operations.

Steps for AI agent (ordered):

1. POST `/api/auth/login` with seller A credentials — capture `sellerToken`.
2. POST `/api/prices` with `Authorization: Bearer {sellerToken}` and body `{"name":"tien dien","unit_price":2500}` — expect 201 and `{ id, name, unit_price, seller_id, createdAt }`.
3. GET `/api/prices` with `Authorization: Bearer {sellerToken}` — expect 200 and list containing the newly created price.
4. POST `/api/auth/login` with seller B credentials — capture `sellerBToken`.
5. GET `/api/prices` with `Authorization: Bearer {sellerBToken}` — expect 200 and list NOT containing seller A's price.
6. POST `/api/prices` with `unit_price: -100` — expect 400.
7. POST `/api/prices` with missing `name` — expect 400.
8. POST `/api/prices` with missing `unit_price` — expect 400.
9. POST `/api/prices` without Authorization header — expect 401.
10. Return structured result: `{"status":"ok|failed","evidence":{...}}`

Machine-readable JSON:

```json
{
  "id": "US-5",
  "title": "Seller creates a price",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "create a price with a name and unit price",
  "so_that": "I can assign it to a house or group for monthly bill calculation",
  "preconditions": [
    "seller is authenticated with valid JWT",
    "application is running"
  ],
  "trigger": "POST /api/prices",
  "acceptance_criteria": [
    "400 on missing or empty name",
    "400 on missing unit_price",
    "400 if unit_price <= 0",
    "201 with { id, name, unit_price, seller_id, createdAt } on success",
    "seller_id set automatically from JWT",
    "401 without valid JWT",
    "403 for buyer role",
    "GET /api/prices scoped to calling seller",
    "newly created price has no house or group assignment yet"
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
      "action": "create price",
      "api": {
        "method": "POST",
        "path": "/api/prices",
        "headers": { "Authorization": "Bearer {sellerToken}" },
        "body": { "name": "tien dien", "unit_price": 2500 }
      },
      "expect": "201 and { id, name, unit_price, seller_id, createdAt }"
    },
    {
      "action": "list seller A's prices",
      "api": {
        "method": "GET",
        "path": "/api/prices",
        "headers": { "Authorization": "Bearer {sellerToken}" }
      },
      "expect": "200 and list containing created price"
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
      "action": "seller B lists prices — must NOT see seller A's price",
      "api": {
        "method": "GET",
        "path": "/api/prices",
        "headers": { "Authorization": "Bearer {sellerBToken}" }
      },
      "expect": "200 and list NOT containing seller A's price"
    },
    {
      "action": "invalid unit_price (negative)",
      "api": {
        "method": "POST",
        "path": "/api/prices",
        "headers": { "Authorization": "Bearer {sellerToken}" },
        "body": { "name": "test", "unit_price": -100 }
      },
      "expect": "400"
    },
    {
      "action": "missing name",
      "api": {
        "method": "POST",
        "path": "/api/prices",
        "headers": { "Authorization": "Bearer {sellerToken}" },
        "body": { "unit_price": 2500 }
      },
      "expect": "400"
    },
    {
      "action": "missing unit_price",
      "api": {
        "method": "POST",
        "path": "/api/prices",
        "headers": { "Authorization": "Bearer {sellerToken}" },
        "body": { "name": "tien dien" }
      },
      "expect": "400"
    },
    {
      "action": "unauthenticated request",
      "api": {
        "method": "POST",
        "path": "/api/prices",
        "body": { "name": "tien dien", "unit_price": 2500 }
      },
      "expect": "401"
    }
  ],
  "data": {
    "price": {
      "id": "uuid",
      "seller_id": "uuid",
      "name": "string",
      "unit_price": "decimal (> 0)",
      "createdAt": "timestamp",
      "updatedAt": "timestamp"
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
    "POST /api/prices": {
      "status": 201,
      "body": {
        "id": "price-uuid",
        "name": "tien dien",
        "unit_price": 2500,
        "seller_id": "seller-a-id",
        "createdAt": "2026-04-03T00:00:00Z"
      }
    },
    "GET /api/prices (sellerA)": {
      "status": 200,
      "body": [
        {
          "id": "price-uuid",
          "name": "tien dien",
          "unit_price": 2500,
          "seller_id": "seller-a-id"
        }
      ]
    },
    "GET /api/prices (sellerB)": {
      "status": 200,
      "body": []
    }
  }
}
```

Notes:

- `seller_id` is set automatically from the JWT — not provided in request body.
- The `name` field is the sole identifier for the price's purpose (e.g., "tien dien", "tien nuoc", "tien phong"). No classification type field is needed.
- After creating a price, assign it to a house or group via separate endpoints:
  - Assign to a group: `POST /api/groups/:id/prices`
  - Assign directly to a house: `POST /api/houses/:id/prices`
- `GET /api/prices` is scoped to the calling seller; the server infers the scope from the JWT — no extra query param needed.
