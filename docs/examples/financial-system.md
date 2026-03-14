# Financial System Example

A realistic BOUND definition for a financial services application with payment processing, account management, and regulatory compliance.

## BOUND Definition

### DANGER ZONES

- `src/payments/calculator.py` — Financial calculations, penny-level precision required
- `src/payments/processor.py` — Payment gateway integration, handles real money
- `migrations/` — Database schema changes, irreversible in production
- `src/auth/` — Authentication and authorization, security-critical
- `config/production.yml` — Production configuration, deployment-critical

### NEVER DO

- Never use `float` for monetary values — always `Decimal` with explicit precision
- Never delete or rename migration files
- Never commit without running the full test suite
- Never hardcode API keys or secrets
- Never bypass rate limiting, even in tests
- Never modify audit log schema without compliance review

### IRON LAWS

- All monetary values use `Decimal` with 2-digit precision
- All API responses include `request_id` field
- Test coverage for payment module never drops below 90%
- All database queries use parameterized statements (no string interpolation)
- Audit trail records every payment state transition
- All external API calls have timeout and retry logic

## Why This BOUND Works

Financial systems have **zero tolerance for certain failures**. A `float` rounding error in payment calculations can compound across millions of transactions. A deleted migration file can corrupt production data.

The BOUND makes these constraints explicit and enforceable. The agent can freely refactor business logic, add features, and optimize performance — as long as it never crosses these lines.

## Key Patterns

1. **Decimal precision** — The single most common financial bug. The IRON LAW makes it impossible to introduce.
2. **Migration safety** — Database schema changes are one-way. The NEVER DO rule prevents accidental deletion.
3. **Audit compliance** — Every payment state change must be logged. The IRON LAW ensures the agent can't skip this.
