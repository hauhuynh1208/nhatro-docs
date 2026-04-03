## Header

Title: Provision initial admin account
ID: US-001
Priority: Must
Actor: admin
As a: admin setting up the system for the first time
I want: the developer to create an admin account using the username and password I provide
So that: I can log in and start operating the system

## Business Value

Without an initial admin account, the admin cannot access the system at all. All downstream management tasks — such as creating seller accounts — are blocked, making the platform inoperable from a management standpoint.

## Preconditions

- Backend and database have been set up

## Trigger

- None (one-time setup action performed by the developer before first use)

## Acceptance Criteria

1. The admin can log in to the system using the username and password provided to the developer (e.g., `usernameAdmin` / `passwordAdmin`).
2. After logging in, the admin has access to admin-level capabilities (e.g., can create seller accounts).

## UI/UX References

- None

---

## Machine-readable (for AI agent — do not edit manually)

```json
{
  "id": "US-001",
  "title": "Provision initial admin account",
  "priority": "Must",
  "actor": "admin",
  "as_a": "admin setting up the system for the first time",
  "i_want": "developer to create an admin account using the username and password I provide",
  "so_that": "I can log in and start operating the system",
  "business_value": "Without an initial admin account, the admin cannot access the system or perform any management tasks such as creating seller accounts.",
  "preconditions": ["Backend and database have been set up"],
  "trigger": "None",
  "acceptance_criteria": [
    "Admin can log in using the username and password provided to the developer.",
    "After login, admin can access admin-level capabilities (e.g., create seller accounts)."
  ],
  "entities_involved": ["admin"],
  "ui_references": []
}
```
