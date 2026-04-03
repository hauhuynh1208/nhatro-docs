# User Story Template

> **Audience:** Written by Product Owner. Read by team members and AI agents.
> **Rule:** Describe _what the user wants and why_ — not how the system implements it.

---

## Header

Title: <short title>
ID: <US-###>
Priority: <Must | Should | Could | Won't>
Actor: <admin | seller | buyer>
As a: <role description>
I want: <goal / action, from user's perspective>
So that: <business benefit / outcome>

## Business Value

<1–2 sentences explaining why this feature matters to the business or product, beyond the individual user's benefit.>

## Preconditions

- <what must be true before the user can perform this action>
- <e.g.: user is logged in, a house record exists, etc.>

## Trigger

- <what the user does to start this flow, described in plain language>
- <e.g.: "User taps 'Issue Bill' on the house detail screen">

## Acceptance Criteria

> Each criterion must be testable and written from the user's or system's observable perspective — no API details.

1. <AC 1 — what the user sees or what the system does>
2. <AC 2 — ...>
3. <AC 3 — ...>

## UI/UX References

- <Figma link or screen name, if available>

---

## Machine-readable (for AI agent — do not edit manually)

> AI agents use this section to break down the story into technical tasks.
> PO fills only the business fields. Technical fields are derived by AI.

```json
{
  "id": "US-###",
  "title": "",
  "priority": "Must | Should | Could | Won't",
  "actor": "",
  "as_a": "",
  "i_want": "",
  "so_that": "",
  "business_value": "",
  "preconditions": [],
  "trigger": "",
  "acceptance_criteria": [],
  "entities_involved": [],
  "ui_references": []
}
```

---

## Example

Title: Seller approves usage and issues monthly bill
ID: US-101
Priority: Must
Actor: seller
As a: seller
I want: review pending utility readings for a house and generate the monthly bill
So that: the buyer receives a correct bill for the current billing period

### Business Value

Accurate billing is the core revenue flow of the platform. Without this feature, sellers cannot charge tenants and the platform has no transaction value.

### Preconditions

- Seller is logged in
- At least one house is managed by this seller
- The house has pending utility readings for the current billing period

### Trigger

- Seller taps "Review & Issue Bill" on the house billing screen

### Acceptance Criteria

1. Seller can see a list of all pending utility readings for the selected house and billing period.
2. Seller can mark individual readings as approved or discarded.
3. After approving, seller can generate a bill — the system calculates the total based on approved readings.
4. If a bill for this house and billing period already exists, the system prevents creating a duplicate and shows a clear message.
5. Once issued, the seller can download or share the bill as a PDF.

### UI/UX References

- Figma: `House Billing Screen v2`

---

### Machine-readable JSON for example

```json
{
  "id": "US-101",
  "title": "Seller approves usage and issues monthly bill",
  "priority": "Must",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "review pending utility readings and generate a monthly bill",
  "so_that": "buyer receives a correct bill for the current billing period",
  "business_value": "Accurate billing is the core revenue flow of the platform",
  "preconditions": [
    "seller is logged in",
    "seller manages at least one house",
    "house has pending utility readings for current billing period"
  ],
  "trigger": "seller taps 'Review & Issue Bill' on house billing screen",
  "acceptance_criteria": [
    "seller can view all pending readings for selected house and billing period",
    "seller can approve or discard individual readings",
    "system calculates bill total from approved readings",
    "duplicate bill for same house and period is blocked with clear message",
    "issued bill can be downloaded or shared as PDF"
  ],
  "entities_involved": ["house", "usage", "bill", "seller"],
  "ui_references": ["House Billing Screen v2"]
}
```

---

> **Note for AI agents:** Use the machine-readable JSON to infer technical tasks (data models, APIs, UI components, business rules). Do not ask the PO for API details — derive them from the entities, acceptance criteria, and error cases above.
