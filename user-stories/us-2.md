Title: Login
ID: US-2
Actor: any registered user (admin, seller, buyer)
As a: registered user
I want: authenticate with email and password
So that: I receive a JWT access token to access protected endpoints

Preconditions:

- User account exists in the `users` table with a bcrypt-hashed password
- Application is running

Trigger:

- User sends `POST /api/auth/login` with email and password

Acceptance Criteria (AC):

1. Successful login returns HTTP 200 with `accessToken` (JWT) and `user` payload (`id`, `email`, `role`).
2. Wrong password or non-existent email returns HTTP 401 — same generic message for both (do not reveal which field failed).
3. Login request body is validated — missing `email` or `password` returns HTTP 400.
4. Successful and failed login attempts are recorded in `audit_log`.

Steps for AI agent (ordered):

1. POST `/api/auth/login` with valid admin credentials — expect 200 and `accessToken` in response.
2. POST `/api/auth/login` with correct email but wrong password — expect 401.
3. POST `/api/auth/login` with non-existent email — expect 401, same message as step 2.
4. POST `/api/auth/login` with missing `password` field — expect 400.
5. Query `audit_logs` table — expect one `login_success` and two `login_failed` entries.

Machine-readable JSON:

```json
{
  "id": "US-2",
  "title": "Login",
  "actor": "registered user",
  "as_a": "registered user",
  "i_want": "authenticate with email and password",
  "so_that": "I receive a JWT to access protected endpoints",
  "preconditions": ["user exists in database", "application running"],
  "trigger": "POST /api/auth/login",
  "acceptance_criteria": [
    "200 with accessToken and user payload on success",
    "401 with generic message on wrong credentials",
    "400 on missing fields",
    "audit_log entry created for success and failure"
  ],
  "steps": [
    {
      "action": "valid login",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "admin@nhatro.com", "password": "Admin@123" }
      },
      "expect": "200 with accessToken"
    },
    {
      "action": "wrong password",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "admin@nhatro.com", "password": "wrong" }
      },
      "expect": "401"
    },
    {
      "action": "non-existent email",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "ghost@nhatro.com", "password": "anything" }
      },
      "expect": "401, same message as wrong password"
    },
    {
      "action": "missing field",
      "api": {
        "method": "POST",
        "path": "/api/auth/login",
        "body": { "email": "admin@nhatro.com" }
      },
      "expect": "400"
    }
  ]
}
```

Notes:

- Do not distinguish between "email not found" and "wrong password" in the error response — always return generic 401.
- JWT secret and expiration are configured via environment variables (`JWT_SECRET`, `JWT_ACCESS_TOKEN_EXPIRATION`).
- Default access token lifetime: 15 minutes.

Steps for AI agent (ordered):

1. Verify auth endpoints exist: POST `/auth/login`, POST `/auth/refresh`, POST `/auth/logout`, POST `/auth/forgot-password`, POST `/auth/reset-password`.
2. Run positive login test: create test user (seller), set password, POST `/auth/login` — expect 200 with token.
3. Run negative login test: wrong password returns 401 and event logged.
4. Test role enforcement: with seller token, attempt POST `/admin/sellers` — expect 403.
5. Test ownership enforcement: seller A attempts to modify seller B's house — expect 403.
6. Test token revocation and refresh behavior.
7. Verify audit logs for login events.

Machine-readable JSON:

```json
{
  "id": "US-3",
  "title": "Authentication & Authorization (login + RBAC)",
  "actor": "system",
  "as_a": "system owner",
  "i_want": "secure auth and role enforcement",
  "so_that": "users can only do allowed actions",
  "preconditions": ["user store", "TLS enabled", "secrets configured"],
  "trigger": "user accesses app or API",
  "acceptance_criteria": [
    "login returns tokens",
    "password hashed",
    "RBAC enforced (401/403)",
    "token revocation works",
    "auth events logged"
  ],
  "steps": [
    {
      "action": "check login endpoint",
      "api": { "method": "POST", "path": "/auth/login" },
      "expect": "200 with token"
    },
    {
      "action": "wrong password",
      "api": {
        "method": "POST",
        "path": "/auth/login",
        "body": { "email": "{email}", "password": "wrong" }
      },
      "expect": "401"
    },
    {
      "action": "role test",
      "api": {
        "method": "POST",
        "path": "/admin/sellers",
        "headers": { "Authorization": "Bearer {seller_token}" }
      },
      "expect": "403"
    },
    {
      "action": "ownership test",
      "api": {
        "method": "PUT",
        "path": "/houses/{other_seller_house_id}",
        "headers": { "Authorization": "Bearer {seller_token}" }
      },
      "expect": "403"
    }
  ]
}
```

Notes / Implementation guidance:

- Use proven libraries for password hashing (bcrypt/argon2) and JWT handling.
- Design RBAC with both role checks and resource ownership checks.
- Consider rate-limiting login attempts and enabling optional 2FA for `admin` role.
- Document token lifetimes and refresh policies in `app-terms.md`.
