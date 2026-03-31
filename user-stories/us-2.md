Title: Developer provisions initial admin account
ID: US-2
Actor: developer
As a: developer / system operator
I want: provision an initial `admin` account and record provenance
So that: there is a trusted `admin` who can create `seller` accounts and operate the system

Preconditions:

- System is deployed and reachable (staging/production environment)
- Developer has access to deployment/ops console or provisioning tool

Trigger:

- Developer runs provisioning script or uses admin console to create the first `admin` account

Acceptance Criteria (AC):

1. A new `admin` record is created with fields: `id`, `email`, `full_name`, `provisioned_by` (developer id or system), `provisioned_at` (timestamp), and `is_active=true`.
2. Provisioning operation is auditable: an `audit_log` entry is created recording `actor=developer`, `action=create_admin`, `target=admin_id`, and timestamp.
3. If provisioning is done via one-time invite link, invite token is created, stored (`invite_token`), single-use, and expires after configurable TTL.
4. Developer receives confirmation (CLI output or email) with next steps; `admin` can sign in (or complete onboarding if invite flow used).
5. If admin email already exists, provisioning fails with clear error (409) and no duplicate admin is created.

Steps for AI agent (ordered):

1. Verify provisioning preconditions (deployment accessible, developer credentials valid).
2. Execute provisioning API or CLI: POST `/ops/admins` with body {"email","full_name","provisioned_by":"developer_id"} — expect 201 and admin payload.
3. Verify `audit_log` entry exists: GET `/audit?target_type=admin&target_id={admin_id}` — expect recent create_admin entry.
4. If invite flow, assert invite email queued with token and simulate onboarding flow (see US-1 onboarding steps).
5. Return structured result: {"status":"ok","admin_id":...,"evidence":{...}}

Machine-readable JSON:

```json
{
  "id": "US-2",
  "title": "Developer provisions initial admin account",
  "actor": "developer",
  "as_a": "developer",
  "i_want": "provision initial admin account",
  "so_that": "admin can manage sellers and system",
  "preconditions": ["deployment accessible", "developer creds valid"],
  "trigger": "ops action",
  "acceptance_criteria": [
    "admin record created with provisioned_by and provisioned_at",
    "audit_log created",
    "invite token single-use if applicable",
    "duplicate email returns 409"
  ],
  "steps": [
    {
      "action": "verify ops access",
      "api": { "method": "GET", "path": "/ops/health" },
      "expect": "200"
    },
    {
      "action": "provision admin",
      "api": {
        "method": "POST",
        "path": "/ops/admins",
        "body": {
          "email": "{email}",
          "full_name": "{name}",
          "provisioned_by": "{developer_id}"
        }
      },
      "expect": "201 and admin payload"
    },
    {
      "action": "check audit",
      "api": {
        "method": "GET",
        "path": "/audit?target_type=admin&target_id={admin_id}"
      },
      "expect": "create_admin entry present"
    }
  ]
}
```

Notes:

- Provisioning is an ops-level operation; implementers should secure the provisioning API behind strong auth (deploy keys or internal network).
- Record `provisioned_by` and `provisioned_at` for traceability.
