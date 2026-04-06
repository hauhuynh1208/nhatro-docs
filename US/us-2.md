## Header

Title: Admin views seller account list
ID: US-002
Priority: Must
Actor: admin
As a: admin
I want: view a list of all created seller accounts
So that: I can monitor and manage sellers in the system

## Business Value

The seller account list gives the admin an overview of all active sellers and serves as the entry point for further management tasks such as viewing details or editing a seller's profile.

## Preconditions

- Admin is logged in

## Trigger

- Admin selects "Sellers" from the side menu

## Acceptance Criteria

1. The list displays each seller with the following information: `full_name`, `phone`, `address`, `created_at`.
2. On desktop, the list is presented as a table.
3. On mobile, each seller is presented as a card.
4. The list is paginated.
5. Each row/card has a checkbox at the beginning (bulk action behaviour to be defined in a separate user story).
6. Clicking a row/card navigates the admin to the seller detail screen.
7. If no sellers exist, the screen displays "No sellers found".

## UI/UX References

- None

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-002",
  "title": "Admin views seller account list",
  "priority": "Must",
  "actor": "admin",
  "as_a": "admin",
  "i_want": "view a list of all created seller accounts",
  "so_that": "I can monitor and manage sellers in the system",
  "business_value": "The seller account list gives the admin an overview of all active sellers and serves as the entry point for further management tasks.",
  "preconditions": ["Admin is logged in"],
  "trigger": "Admin selects 'Sellers' from the side menu",
  "acceptance_criteria": [
    "List displays each seller: full_name, phone, address, created_at.",
    "Desktop: table layout.",
    "Mobile: card layout.",
    "List is paginated.",
    "Each row/card has a checkbox (bulk action to be defined in a separate US).",
    "Clicking a row/card navigates to the seller detail screen.",
    "No sellers: display 'No sellers found'."
  ],
  "entities_involved": ["admin", "seller"],
  "ui_references": []
}
```
