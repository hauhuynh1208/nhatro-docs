# GitHub Copilot Instructions

## Context: Always review app documentation

Before answering any question or generating any output, review the documents in the `apps/` folder to understand the application domain:

- `apps/app-introduction.md` — overview of the app
- `apps/app-actors.md` — who the actors/users are (admin, seller, buyer)
- `apps/app-flow.md` — key user flows
- `apps/app-terms.md` — domain terminology and definitions

Use this context to ensure answers are consistent with the product's domain, terminology, and actor roles.

---

## Writing User Stories

When asked to write or create a User Story, strictly follow the template defined in `US/us-template.md`:

- Use the exact sections and order defined in the template
- Fill in the machine-readable JSON block completely
- Write Acceptance Criteria from the user's observable perspective — no API details, no technical implementation
- Use actor names and terminology consistent with `apps/app-actors.md` and `apps/app-terms.md`
