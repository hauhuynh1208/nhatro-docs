Title: Seller creates house group
ID: US-X
Actor: seller
As a: seller
I want: create a group by providing a name, so that I can later assign multiple houses, prices, and a formula to that group
So that: all houses belonging to the same group share the same electricity/water prices and billing formula, reducing repetitive configuration

Preconditions:

- Caller is authenticated as `seller` with a valid JWT
- Application is running

Trigger:

- Seller sends `POST /api/groups` with `Authorization: Bearer {sellerToken}` and body `{ name, type: 1 }`

Acceptance Criteria (AC):

1. Request body is validated — `name` is required and must be a non-empty string; `type` is required and must be a valid integer (currently only `1` is accepted); missing or invalid fields return HTTP 400.
2. On success, a group record is created with `seller_id = caller.id` and `type = 1`; HTTP 201 is returned with `{ id, name, type, seller_id, createdAt }`.
3. Requests without a valid JWT return HTTP 401.
4. A `buyer` or any role other than `seller` / `admin` calling this endpoint returns HTTP 403.
5. `GET /api/groups` returns only groups belonging to the calling seller (`seller_id = caller.id`) — groups created by other sellers are not visible.
6. Two groups belonging to the same seller can have the same name (no uniqueness constraint on name per seller). _(Names are for display only; identity is by `id`.)_
7. A newly created group has no houses, prices, or formula assigned yet — those are separate operations.

Steps for AI agent (ordered):

1. POST `/api/auth/login` with seller credentials — capture `sellerToken`.
2. POST `/api/groups` with `Authorization: Bearer {sellerToken}` and body `{"name":"Dãy trọ A","type":1}` — expect 201 and `{ id, name, type, seller_id, createdAt }`.
3. GET `/api/groups` with `Authorization: Bearer {sellerToken}` — expect 200 and list containing the newly created group.
4. POST `/api/auth/login` with a different seller's credentials — capture `sellerBToken`.
5. GET `/api/groups` with `Authorization: Bearer {sellerBToken}` — expect 200 and list NOT containing the group from step 2.
6. POST `/api/groups` with `Authorization: Bearer {sellerToken}` and body `{}` (missing name) — expect 400.
7. POST `/api/groups` without Authorization header — expect 401.
8. Return structured result: `{"status":"ok|failed","evidence":{...}}`

Machine-readable JSON:

```json
{
  "id": "US-X",
  "title": "Seller creates house group",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "create a named group to organize houses with shared prices and formula",
  "so_that": "houses in the same group share electricity/water prices and billing formula",
  "preconditions": [
    "seller is authenticated with valid JWT",
    "application is running"
  ],
  "trigger": "POST /api/groups",
  "acceptance_criteria": [
    "400 on missing or empty name, or missing/invalid type",
    "201 with { id, name, type, seller_id, createdAt } on success",
    "type=1 means house group; future type=2 will be user group",
    "401 without valid JWT",
    "403 for non-seller/admin roles",
    "GET /api/groups scoped to calling seller — other sellers' groups not visible",
    "newly created group has no houses/prices/formula assigned yet"
  ],
  "steps": [
    {
      "action": "seller login",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "sellerA@test.com", "password": "Test@1234" }
      },
      "expect": "200 and accessToken"
    },
    {
      "action": "create group",
      "api": {
        "method": "POST",
        "path": "/api/groups",
        "headers": { "Authorization": "Bearer {sellerToken}" },
        "body": { "name": "Dãy trọ A", "type": 1 }
      },
      "expect": "201 and { id, name, type, seller_id, createdAt }"
    },
    {
      "action": "list seller's groups",
      "api": {
        "method": "GET",
        "path": "/api/groups",
        "headers": { "Authorization": "Bearer {sellerToken}" }
      },
      "expect": "200 and list containing created group"
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
      "action": "seller B lists groups — must NOT see seller A's group",
      "api": {
        "method": "GET",
        "path": "/api/groups",
        "headers": { "Authorization": "Bearer {sellerBToken}" }
      },
      "expect": "200 and list NOT containing seller A's group"
    },
    {
      "action": "missing name validation",
      "api": {
        "method": "POST",
        "path": "/api/groups",
        "headers": { "Authorization": "Bearer {sellerToken}" },
        "body": {}
      },
      "expect": "400"
    },
    {
      "action": "unauthenticated request",
      "api": {
        "method": "POST",
        "path": "/api/groups",
        "body": { "name": "Test" }
      },
      "expect": "401"
    }
  ],
  "data": {
    "group": {
      "id": "uuid",
      "seller_id": "uuid",
      "name": "string",
      "type": "int (1=house_group, 2=user_group)",
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
    "POST /api/groups": {
      "status": 201,
      "body": {
        "id": "group-uuid",
        "name": "Dãy trọ A",
        "type": 1,
        "seller_id": "seller-a-id",
        "createdAt": "2026-04-03T00:00:00Z"
      }
    },
    "GET /api/groups (sellerA)": {
      "status": 200,
      "body": [
        {
          "id": "group-uuid",
          "name": "Dãy trọ A",
          "type": 1,
          "seller_id": "seller-a-id"
        }
      ]
    },
    "GET /api/groups (sellerB)": {
      "status": 200,
      "body": []
    }
  }
}
```

Notes:

- `seller_id` is set automatically from the JWT — not provided in request body.
- `type` distinguishes the group's purpose: `1` = house group (current), `2` = user group (future). Only `type=1` is accepted for now; the API is designed to be extended without breaking changes.
- When `type` is missing or not a valid value, return HTTP 400.
- `group` is a container for houses. After creation, separate operations are used to:
  - Assign prices to the group: `POST /api/groups/:id/prices`
  - Assign a formula to the group: `POST /api/groups/:id/formula`
  - Add a house to the group: `PATCH /api/houses/:id` with `{ group_id }`
- When a house belongs to a group, it inherits the group's prices and formula for billing. Direct per-house price/formula assignments are overridden by the group's configuration.
- A house can only belong to one group at a time (`group_id` is a single nullable FK on `house`).
- `GET /api/groups` is scoped to the calling seller; server infers the scope from the JWT — no extra query param needed.
