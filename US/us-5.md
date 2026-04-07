## Header

Title: Seller creates a buyer
ID: US-005
Priority: Must
Actor: seller
As a: seller
I want: create a new buyer with their name, room price, number of occupants, and initial utility readings
So that: the buyer is registered in the system and ready to be billed in future billing cycles

## Business Value

Creating a buyer is the foundational step for billing. Without a buyer record, the seller cannot collect utility readings, issue bills, or track payments for that unit.

## Preconditions

- Seller is logged in

## Trigger

- Seller taps "Add Buyer" on the buyer list screen

## Acceptance Criteria

1. The creation form includes the following fields: name (required), number of occupants — `people_count` (required, positive integer), room price — `room_price` (required, positive number), electricity baseline (required, non-negative number), water baseline (required, non-negative number), baseline month (required, YYYY-MM format).
2. Submitting the form with any required field empty shows a validation error on that field.
3. Submitting with a non-positive value for `people_count` or `room_price` shows a validation error.
4. On successful submission, the new buyer appears in the buyer list.
5. If seller later updates `room_price` or `people_count` for the buyer, previously issued bills for that buyer remain unchanged.

## UI/UX References

- None

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-005",
  "title": "Seller creates a buyer",
  "priority": "Must",
  "actor": "seller",
  "as_a": "seller",
  "i_want": "create a new buyer with their name, room price, number of occupants, and initial utility readings",
  "so_that": "the buyer is registered in the system and ready to be billed in future billing cycles",
  "business_value": "Creating a buyer is the foundational step for billing. Without a buyer record, the seller cannot collect utility readings, issue bills, or track payments for that unit.",
  "preconditions": ["Seller is logged in"],
  "trigger": "Seller taps 'Add Buyer' on the buyer list screen",
  "acceptance_criteria": [
    "Form includes: name (required), people_count (required, positive integer), room_price (required, positive number), electricity_baseline (required, non-negative), water_baseline (required, non-negative), baseline_month (required, YYYY-MM).",
    "Submitting with any required field empty shows a validation error on that field.",
    "Submitting with a non-positive value for people_count or room_price shows a validation error.",
    "On success, the new buyer appears in the buyer list.",
    "If seller later updates room_price or people_count, previously issued bills remain unchanged."
  ],
  "entities_involved": ["seller", "buyer"],
  "ui_references": []
}
```
