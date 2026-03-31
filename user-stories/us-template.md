User Story Template (Human-readable + Machine-readable)

---

Title: <short title>
ID: <US-###>
Actor: <admin|seller|buyer>
As a: <role description>
I want: <goal / action>
So that: <benefit / outcome>

Preconditions:

- <system state required before story runs>

Trigger:

- <what starts the flow (user action, cron, webhook)>

Acceptance Criteria (AC):

1. <AC 1 — specific, testable>
2. <AC 2 — specific, testable>
3. <AC 3 — specific, testable>

Steps for AI agent (ordered):

1. Validate preconditions.
2. Call API / perform action X with payload Y.
3. Verify response and resulting system state.
4. Return structured result: {"status":"ok|failed","details":..., "evidence":...}

Machine-readable section (JSON):

```json
{
  "id": "US-001",
  "title": "",
  "actor": "",
  "as_a": "",
  "i_want": "",
  "so_that": "",
  "preconditions": [],
  "trigger": "",
  "acceptance_criteria": [],
  "steps": [
    {
      "action": "",
      "api": { "method": "", "path": "", "body": {} },
      "expect": ""
    }
  ],
  "data": {},
  "mocks": {}
}
```

Example user story (complete, executable):

Title: Seller approves usage and issues monthly bill
ID: US-101
Actor: seller
As a: seller
I want: review pending usage records for a house, approve them, and generate the monthly bill
So that: buyer receives a correct bill reflecting approved usage

Preconditions:

- seller account is authenticated
- house exists and has at least one pending usage for current billing_cycle

Trigger:

- seller clicks "Review & Issue Bill" on house billing screen

Acceptance Criteria:

1. The system lists all pending `usage` records for the specified `house` and `billing_cycle`.
2. When seller marks usages as `approved`, each usage status becomes `approved` with `approved_by` and `approved_at` recorded.
3. System computes consumption = current_reading - previous_reading for each approved usage, applies assigned `price` and `formula`, and creates a single `bill` for that `house` and `billing_cycle`.
4. If a `bill` already exists for (house, billing_cycle), the action fails with a clear error.
5. The created `bill` has `status=issued`, contains line_items matching computed amounts, and an `export_url` (PDF) is returned.

Steps for AI agent (ordered):

1. Verify seller auth (token valid).
2. GET `/houses/{house_id}/usages?billing_cycle=YYYY-MM&status=pending` — expect 200 and non-empty list.
3. For each usage id in list, POST `/usages/{usage_id}/approve` with body {"approved_by":"seller_id"} — expect 200 and status updated.
4. POST `/houses/{house_id}/bills` with body {"billing_cycle":"YYYY-MM","issued_by":"seller_id"} — expect 201 and bill payload.
5. Verify bill uniqueness: GET `/houses/{house_id}/bills?billing_cycle=YYYY-MM` returns single bill.
6. Return result JSON with bill id, total_amount, and `export_url`.

Machine-readable JSON for example:

```json
{
  "id": "US-101",
  "title": "Seller approves usage and issues monthly bill",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "approve usages and generate bill",
  "so_that": "buyer receives correct bill",
  "preconditions": ["auth_token valid", "house exists", "pending usages exist"],
  "trigger": "seller action",
  "acceptance_criteria": [
    "list pending usages returned",
    "each usage status updated to approved",
    "single bill created for (house,billing_cycle)",
    "export_url present"
  ],
  "steps": [
    { "action": "verify auth", "api": null, "expect": "200 / valid token" },
    {
      "action": "list usages",
      "api": {
        "method": "GET",
        "path": "/houses/{house_id}/usages?billing_cycle={cycle}&status=pending"
      },
      "expect": "200 and list"
    },
    {
      "action": "approve usage",
      "api": {
        "method": "POST",
        "path": "/usages/{usage_id}/approve",
        "body": { "approved_by": "{seller_id}" }
      },
      "expect": "200"
    },
    {
      "action": "create bill",
      "api": {
        "method": "POST",
        "path": "/houses/{house_id}/bills",
        "body": { "billing_cycle": "{cycle}", "issued_by": "{seller_id}" }
      },
      "expect": "201 and bill payload"
    }
  ],
  "data": {
    "usage": {
      "id": "int",
      "house_id": "int",
      "current_reading": "number",
      "previous_reading": "number",
      "status": "pending|approved|discarded"
    },
    "bill": {
      "id": "int",
      "house_id": "int",
      "billing_cycle": "YYYY-MM",
      "total_amount": "decimal",
      "status": "issued"
    }
  },
  "mocks": {
    "/auth/validate": { "status": 200, "body": { "user_id": "seller_id" } },
    "/houses/1/usages": {
      "status": 200,
      "body": [
        {
          "id": 11,
          "current_reading": 1500,
          "previous_reading": 1400,
          "status": "pending"
        }
      ]
    }
  }
}
```

Notes for implementer / AI agent:

- All API paths and payloads are examples — adapt to actual API surface.
- AI should always return structured JSON (status, evidence, errors) after executing steps.
- Acceptance Criteria must be fully satisfied for the story to be marked done.

---

Use this template for all user stories in this folder. Keep the machine-readable JSON updated so automation agents can parse and run them.
