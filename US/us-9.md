## Header

Title: Seller creates a formula
ID: US-009
Priority: Must
Actor: seller
As a: seller
I want: create a formula using a structured builder that supports variables, fixed numbers, nested formulas, and if/else conditions
So that: I have a reusable billing expression that can be assigned to buyers or used in sheet configs

## Business Value

Formulas are the core of the billing calculation engine. A structured builder prevents syntax errors and ensures every formula is well-formed, while support for if/else blocks and formula references enables complex, real-world billing logic.

## Preconditions

- Seller is logged in

## Trigger

- Seller taps "Add Formula" on the formula list screen

## Acceptance Criteria

1. The creation form includes a `name` field (required).
2. Submitting with a name already used by another formula of this seller shows a validation error.
3. The builder allows the seller to construct the formula output by combining operands with arithmetic operators (`+`, `-`, `*`, `/`).
4. Each operand can be one of four types: a **fixed number**, a **variable** (selected from the seller's variable list), another **formula** (selected from the seller's formula list), or an **if/else block**.
5. An if/else block consists of: a condition (left operand, a comparison operator — `>`, `<`, `>=`, `<=`, `==`, `!=` — and right operand), a `then` expression, and an `else` expression. Each of `then` and `else` is itself a full expression, allowing further nesting of if/else blocks or any other operand type.
6. When selecting a formula as an operand, the list excludes the formula currently being created and any formula that would introduce a circular reference.
7. On successful submission, the new formula appears in the formula list.

## UI/UX References

- None

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-009",
  "title": "Seller creates a formula",
  "priority": "Must",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "create a formula using a structured builder that supports variables, fixed numbers, nested formulas, and if/else conditions",
  "so_that": "I have a reusable billing expression that can be assigned to buyers or used in sheet configs",
  "business_value": "Formulas are the core of the billing calculation engine. A structured builder prevents syntax errors and ensures every formula is well-formed, while support for if/else blocks and formula references enables complex, real-world billing logic.",
  "preconditions": ["Seller is logged in"],
  "trigger": "Seller taps 'Add Formula' on the formula list screen",
  "acceptance_criteria": [
    "Form includes: name (required).",
    "Submitting with a name already used by another formula of this seller shows a validation error.",
    "Builder allows combining operands with arithmetic operators: +, -, *, /.",
    "Each operand can be: a fixed number, a variable (from seller's list), another formula (from seller's list), or an if/else block.",
    "An if/else block has: a condition (left operand, comparison operator >, <, >=, <=, ==, !=, right operand), a then expression, and an else expression — each of which is a full expression allowing further nesting.",
    "Formula selection list excludes the current formula and any formula that would create a circular reference.",
    "On success, the new formula appears in the formula list."
  ],
  "entities_involved": ["seller", "formula", "variable"],
  "ui_references": []
}
```
