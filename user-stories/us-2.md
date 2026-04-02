Title: Authentication & Authorization (login + RBAC)
ID: US-3
Actor: system / admin
As a: system owner
I want: authentication and role-based authorization implemented for `admin`, `seller`, and `buyer`
So that: only authenticated users perform actions allowed by their role

Preconditions:

- System has user store (users table) and secure secrets management for password hashing and tokens
- TLS enabled for transport

Trigger:

- A user attempts to access the application UI or API (login flow or token-present request)

Acceptance Criteria (AC):

1. Login: users can authenticate via email+password; successful login returns short-lived access token (JWT) and refresh token if applicable.
2. Passwords are stored as salted hashes; password reset flow via email token is available.
3. RBAC: `admin`, `seller`, `buyer` roles are enforced — endpoints and UI actions check role and ownership (e.g., seller can only manage their houses).
4. Protected endpoints return 401 for unauthenticated and 403 for unauthorized actions.
5. Sessions/tokens can be revoked (logout) and refresh tokens are single-use rotate-on-refresh.
6. Auth-related events (login, failed login, password reset, token revoke) are recorded in `audit_log` or auth-specific logs.

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
