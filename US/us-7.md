## Header

Title: Seller creates a variable
ID: US-007
Priority: Must
Actor: seller
As a: seller
I want: create a new variable by entering a name
So that: I can reference it in formulas to build billing expressions

## Business Value

Variables are the named building blocks of formulas. Allowing sellers to define and name variables independently makes formulas reusable and easier to maintain across different buyers and billing configurations.

## Preconditions

- Seller is logged in

## Trigger

- Seller taps "Add Variable" on the variable list screen

## Acceptance Criteria

1. The creation form includes the following field: `name` (required).
2. Submitting the form with `name` empty shows a validation error on that field.
3. Submitting with a name already used by another variable of this seller shows a validation error.
4. On successful submission, the new variable appears in the variable list.

## UI/UX References

- None

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-007",
  "title": "Seller creates a variable",
  "priority": "Must",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "create a new variable by entering a name",
  "so_that": "I can reference it in formulas to build billing expressions",
  "business_value": "Variables are the named building blocks of formulas. Allowing sellers to define and name variables independently makes formulas reusable and easier to maintain across different buyers and billing configurations.",
  "preconditions": ["Seller is logged in"],
  "trigger": "Seller taps 'Add Variable' on the variable list screen",
  "acceptance_criteria": [
    "Form includes: name (required).",
    "Submitting with name empty shows a validation error on that field.",
    "Submitting with a name already used by another variable of this seller shows a validation error.",
    "On success, the new variable appears in the variable list."
  ],
  "entities_involved": ["seller", "variable"],
  "ui_references": []
}
```
