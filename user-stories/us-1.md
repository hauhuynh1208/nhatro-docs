Title: Create initial admin account via seed script
ID: US-2
Actor: application admin
As a: application admin
I want: run a seed script to create the initial admin account with a predefined email and password
So that: an admin account exists in the database and can be used to sign in and operate the system

Preconditions:

- Database is running and migrations have been applied
- `.env` is configured with valid database credentials

Trigger:

- Admin runs `pnpm seed` (or `pnpm db:setup`) from the project root

Acceptance Criteria (AC):

1. Running the seed script creates an admin user in the `users` table with a predefined email, a bcrypt-hashed password, `role=admin`, and `isActive=true`.
2. If the admin email already exists, the script skips creation and exits cleanly with an informational message — no duplicate is created.
3. The script prints the admin credentials (email and default password) and a reminder to change the password after first login.

Steps for AI agent (ordered):

1. Run `pnpm seed`.
2. Query the `users` table — expect exactly one row with `email=admin@nhatro.com`, `role=admin`, `isActive=true`.
3. Re-run `pnpm seed` — expect "Admin user already exists. Skipping seed." with exit code 0 and still exactly one admin row.
4. Run `./scripts/verify-db.sh` — expect all checks to pass.

Machine-readable JSON:

```json
{
  "id": "US-2",
  "title": "Create initial admin account via seed script",
  "actor": "application admin",
  "as_a": "application admin",
  "i_want": "seed script to create admin account",
  "so_that": "admin exists in database and can sign in",
  "preconditions": [
    "database running",
    "migrations applied",
    ".env configured"
  ],
  "trigger": "pnpm seed",
  "acceptance_criteria": [
    "admin row created with predefined email, hashed password, role=admin, isActive=true",
    "duplicate run skips creation cleanly",
    "credentials printed to CLI with password-change reminder"
  ],
  "steps": [
    {
      "action": "run seed",
      "cli": "pnpm seed",
      "expect": "exit 0, admin created message"
    },
    {
      "action": "verify admin row",
      "db": "SELECT email, role, isActive FROM users WHERE email = 'admin@nhatro.com'",
      "expect": "one row, role=admin, isActive=true"
    },
    {
      "action": "re-run seed (idempotency)",
      "cli": "pnpm seed",
      "expect": "exit 0, skipping message, still one admin row"
    },
    {
      "action": "run verification script",
      "cli": "./scripts/verify-db.sh",
      "expect": "all checks passed"
    }
  ]
}
```

Notes:

- Default credentials are `admin@nhatro.com` / `Admin@123` — change after first login.
- The seed script is idempotent: safe to run multiple times.
- For production, replace the predefined password via environment variable or a post-seed password-reset flow.
