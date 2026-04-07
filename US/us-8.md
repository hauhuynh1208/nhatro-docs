## Header

Title: Seller views formula list
ID: US-008
Priority: Must
Actor: seller
As a: seller
I want: view a list of all formulas I have created
So that: I can monitor my formulas and navigate to their details for further management

## Business Value

Formulas define how bills are calculated for each buyer. A clear formula list gives sellers an overview of all billing expressions and a starting point to review or update them.

## Preconditions

- Seller is logged in

## Trigger

- Seller selects "Formulas" from the side menu

## Acceptance Criteria

1. The list displays each formula with the following information: `name`, `created_at`.
2. On desktop, the list is presented as a table.
3. On mobile, each formula is presented as a card.
4. The list is paginated.
5. Each row/card has a checkbox (bulk action behaviour to be defined in a separate user story).
6. The formula's `name` in each row/card is a link. Clicking it navigates the seller to the formula detail screen.
7. If no formulas exist, the screen displays "No formulas found".

## UI/UX References

- None

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-008",
  "title": "Seller views formula list",
  "priority": "Must",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "view a list of all formulas I have created",
  "so_that": "I can monitor my formulas and navigate to their details for further management",
  "business_value": "Formulas define how bills are calculated for each buyer. A clear formula list gives sellers an overview of all billing expressions and a starting point to review or update them.",
  "preconditions": ["Seller is logged in"],
  "trigger": "Seller selects 'Formulas' from the side menu",
  "acceptance_criteria": [
    "List displays each formula: name, created_at.",
    "Desktop: table layout.",
    "Mobile: card layout.",
    "List is paginated.",
    "Each row/card has a checkbox (bulk action to be defined in a separate US).",
    "The formula's name in each row/card is a link. Clicking it navigates to the formula detail screen.",
    "No formulas: display 'No formulas found'."
  ],
  "entities_involved": ["seller", "formula"],
  "ui_references": []
}
```
