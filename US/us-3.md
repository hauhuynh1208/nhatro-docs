## Header

Title: Admin creates seller account
ID: US-003
Priority: Must
Actor: admin
As a: admin
I want: create a seller account by providing username, password, and optional profile information
So that: the seller can log in and start managing their rental properties on the system

## Business Value

Seller account is a prerequisite for any seller to use the system. Without this feature, the admin cannot onboard sellers, which blocks the entire rental management flow — from creating houses to issuing bills.

## Preconditions

- Admin is logged in

## Trigger

- Admin taps "Create Seller" on the seller accounts management screen

## Acceptance Criteria

1. The screen displays a form with the following fields: `username` (required), `password` (required), `full_name` (optional), `phone` (optional), `address` (optional), `email` (optional).
2. If `username` or `password` is left empty, the form shows a validation error on the respective field and prevents submission.
3. If `password` is fewer than 6 characters, the form shows a validation error on the `password` field and prevents submission.
4. If the submitted `username` already exists in the system, the form shows an error message: "Username already exists."
5. If the account is created successfully, a toast message "Seller account created successfully" is shown and the admin is redirected to the seller accounts management screen.

## UI/UX References

- None

## Notes

- Communicating the credentials (username and password) to the seller is the admin's responsibility and is out of scope for this user story.

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-003",
  "title": "Admin creates seller account",
  "priority": "Must",
  "actor": "admin",
  "as_a": "admin",
  "i_want": "create a seller account by providing username, password, and optional profile information",
  "so_that": "the seller can log in and start managing their rental properties on the system",
  "business_value": "Seller account is a prerequisite for any seller to use the system. Without this feature, the entire rental management flow is blocked.",
  "preconditions": ["Admin is logged in"],
  "trigger": "Admin taps 'Create Seller' on the seller accounts management screen",
  "acceptance_criteria": [
    "Form displays: username (required), password (required), full_name (optional), phone (optional), address (optional), email (optional).",
    "Empty username or password shows field-level validation error and blocks submission.",
    "Password shorter than 6 characters shows validation error on the password field and blocks submission.",
    "Duplicate username shows error: 'Username already exists.'",
    "On success: toast 'Seller account created successfully' and redirect to seller accounts management screen."
  ],
  "entities_involved": ["admin", "seller"],
  "ui_references": []
}
```
