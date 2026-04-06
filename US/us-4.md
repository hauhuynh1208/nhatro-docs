## Header

Title: Seller views buyer list
ID: US-004
Priority: Must
Actor: seller
As a: seller
I want: view a list of all buyers I have created
So that: I can monitor my buyers and navigate to their details for further management

## Business Value

The buyer list is the seller's central view for managing all billing units. Without this screen, sellers cannot navigate to individual buyer records to assign formulas, review submissions, or issue bills.

## Preconditions

- Seller is logged in

## Trigger

- Seller selects "Buyers" from the side menu

## Acceptance Criteria

1. The list displays each buyer with the following information: `full_name`, `phone`, `baseline_month`, `created_at`.
2. On desktop, the list is presented as a table.
3. On mobile, each buyer is presented as a card.
4. The list is paginated.
5. Each row/card has a checkbox (bulk action behaviour to be defined in a separate user story).
6. Clicking a row/card navigates the seller to the buyer detail screen.
7. If no buyers exist, the screen displays "No buyers found".

## UI/UX References

- None

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-004",
  "title": "Seller views buyer list",
  "priority": "Must",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "view a list of all buyers I have created",
  "so_that": "I can monitor my buyers and navigate to their details for further management",
  "business_value": "The buyer list is the seller's central view for managing all billing units. Without this screen, sellers cannot navigate to individual buyer records to assign formulas, review submissions, or issue bills.",
  "preconditions": ["Seller is logged in"],
  "trigger": "Seller selects 'Buyers' from the side menu",
  "acceptance_criteria": [
    "List displays each buyer: full_name, phone, baseline_month, created_at.",
    "Desktop: table layout.",
    "Mobile: card layout.",
    "List is paginated.",
    "Each row/card has a checkbox (bulk action to be defined in a separate US).",
    "Clicking a row/card navigates to the buyer detail screen.",
    "No buyers: display 'No buyers found'."
  ],
  "entities_involved": ["seller", "buyer"],
  "ui_references": []
}
```
