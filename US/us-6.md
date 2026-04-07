## Header

Title: Seller views variable list
ID: US-006
Priority: Must
Actor: seller
As a: seller
I want: view a list of all variables I have created
So that: I can monitor my variables and see which formulas are currently using each one

## Business Value

Variables are the building blocks of formulas. Having a clear overview of all variables — including which formulas reference them — helps sellers manage their billing configuration and avoid creating duplicates.

## Preconditions

- Seller is logged in

## Trigger

- Seller selects "Variables" from the side menu

## Acceptance Criteria

1. The list displays each variable with the following information: `name`, `description`, list of formula names that reference this variable, `created_at`.
2. If a variable is not referenced by any formula, that column displays "Not used in any formula".
3. On desktop, the list is presented as a table.
4. On mobile, each variable is presented as a card.
5. The list is paginated.
6. Each row/card has a checkbox (bulk action behaviour to be defined in a separate user story).
7. Clicking a row/card navigates the seller to the variable detail screen.
8. If no variables exist, the screen displays "No variables found".

## UI/UX References

- None

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-006",
  "title": "Seller views variable list",
  "priority": "Must",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "view a list of all variables I have created",
  "so_that": "I can monitor my variables and see which formulas are currently using each one",
  "business_value": "Variables are the building blocks of formulas. Having a clear overview of all variables — including which formulas reference them — helps sellers manage their billing configuration and avoid creating duplicates.",
  "preconditions": ["Seller is logged in"],
  "trigger": "Seller selects 'Variables' from the side menu",
  "acceptance_criteria": [
    "List displays each variable: name, description, list of formula names referencing this variable, created_at.",
    "If a variable is not referenced by any formula, that column displays 'Not used in any formula'.",
    "Desktop: table layout.",
    "Mobile: card layout.",
    "List is paginated.",
    "Each row/card has a checkbox (bulk action to be defined in a separate US).",
    "Clicking a row/card navigates to the variable detail screen.",
    "No variables: display 'No variables found'."
  ],
  "entities_involved": ["seller", "variable", "formula"],
  "ui_references": []
}
```
