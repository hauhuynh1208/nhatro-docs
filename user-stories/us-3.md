Title: Admin creates seller account
ID: US-1
Actor: admin
As a: admin
I want: create a seller account (with email and contact) and invite the seller to set up login
So that: sellers can sign in and start creating `house`, `price`, and `formula`

Preconditions:

- admin account exists and is authenticated (admin accounts are provisioned by developer)
- system email service is configured to send invite emails (or developer will provision temporary credentials)

Trigger:

- admin clicks "Create Seller" in admin console and submits seller details

Acceptance Criteria (AC):

1. The create-seller form validates required fields: email (format), full_name, phone (optional).
2. On success, a `seller` record is created with `status=pending` or `active` (depending on invite flow), and `created_by=admin_id` is recorded.
3. An invitation email (or setup link) is sent to the seller email containing a one-time setup token.
4. Seller can follow the link to set password and complete onboarding; after onboarding, seller can authenticate and access seller dashboard.
5. If email already exists, API returns 409 conflict and no duplicate `seller` is created.

Steps for AI agent (ordered):

1. Verify admin auth (token valid).
2. Validate payload fields locally.
3. POST `/admin/sellers` with body {"email","full_name","phone","created_by":"{admin_id}"} — expect 201 and seller payload (id, status).
4. Confirm an invite email was queued/sent (GET `/jobs/email?recipient={email}` or check response `invite_sent=true`).
5. Simulate seller onboarding: GET invite link from email mock, POST `/auth/set-password` with token and password — expect 200 and ability to login.
6. Verify seller can authenticate: POST `/auth/login` with email/password — expect 200 and auth token.
7. Return structured result: {"status":"ok","seller_id":...,"evidence":{...}}

Machine-readable JSON:

```json
{
  "id": "US-1",
  "title": "Admin creates seller account",
  "actor": "admin",
  "as_a": "admin",
  "i_want": "create a seller account and send invite",
  "so_that": "seller can sign in and manage houses",
  "preconditions": ["admin authenticated", "email service configured"],
  "trigger": "admin action",
  "acceptance_criteria": [
    "form validation",
    "seller record created with created_by",
    "invite email queued/sent",
    "seller can complete onboarding and login",
    "duplicate email returns 409"
  ],
  "steps": [
    {
      "action": "verify admin auth",
      "api": { "method": "GET", "path": "/auth/validate" },
      "expect": "200"
    },
    {
      "action": "create seller",
      "api": {
        "method": "POST",
        "path": "/admin/sellers",
        "body": {
          "email": "{email}",
          "full_name": "{name}",
          "phone": "{phone}",
          "created_by": "{admin_id}"
        }
      },
      "expect": "201 and seller payload"
    },
    {
      "action": "check invite email",
      "api": { "method": "GET", "path": "/jobs/email?recipient={email}" },
      "expect": "invite link present"
    },
    {
      "action": "seller set password",
      "api": {
        "method": "POST",
        "path": "/auth/set-password",
        "body": { "token": "{token}", "password": "{pwd}" }
      },
      "expect": "200"
    },
    {
      "action": "seller login",
      "api": {
        "method": "POST",
        "path": "/auth/login",
        "body": { "email": "{email}", "password": "{pwd}" }
      },
      "expect": "200 and token"
    }
  ],
  "data": {
    "seller": {
      "id": "int",
      "email": "string",
      "status": "pending|active",
      "created_by": "int"
    }
  },
  "mocks": {
    "/jobs/email?recipient=test@example.com": {
      "status": 200,
      "body": [
        {
          "subject": "Invite",
          "body": "https://app.example.com/onboard?token=abc123"
        }
      ]
    }
  }
}
```

Notes:

- Decide whether new sellers are `pending` (require completing invite) or `active` immediately.
- Invite token must expire and be single-use; record `invite_token`, `invite_expires_at` on seller record.
