# ADR-010: Multi-tenancy-ready schema via `organization_id`

**Status:** Accepted

## Context
EAOS is being built single-tenant (one company, per an earlier project decision) but with an
explicit intent to support multiple client organizations later. The question: do we design the
schema for multi-tenancy now, or add it when it's actually needed?

## Alternatives Considered

1. **Design fully multi-tenant now** (separate schema-per-tenant, or a fully built tenant-routing layer)
   Rejected — this is real complexity (tenant provisioning, cross-tenant query isolation, routing
   logic) with zero current users to justify it. Building it now means learning and maintaining
   machinery that does nothing yet.

2. **Ignore multi-tenancy entirely, retrofit later if needed**
   Rejected — adding a tenant identifier to existing tables after they're populated with real data
   is a genuinely painful migration: every foreign key, every query, every index potentially needs
   revisiting, and you risk data ambiguity (which rows belong to which org) if it wasn't captured
   at write-time.

3. **Add `organization_id` as a column on every core table now, without building tenant-routing logic**
   Selected.

## Decision
Every core table (`users`, `tasks`, `workflows`, `automations`, `logs`, and others added later)
includes an `organization_id UUID` foreign key referencing an `organizations` table, from the very
first migration. A single row exists in `organizations` for now (the one company this platform
serves). No tenant-routing, no per-tenant query scoping, no tenant-aware auth — none of that is
built yet.

## Reasoning
- **Cost asymmetry**: adding one column and one foreign key now costs minutes. Retrofitting it
  onto live data later costs a migration, a backfill strategy, and real risk of misattributing
  existing rows.
- **No behavioral complexity added**: because there's only ever one `organization_id` value right
  now, every query still works exactly as it would without this column — it's inert until it's needed.
- **Keeps the option open without paying for it**: when multi-tenancy is actually needed, the
  schema is already correct; only the *application layer* (auth scoping, query filters, tenant
  provisioning) needs to be built — a much smaller, well-contained piece of work than a full
  schema migration.

## Consequences
- Every new table added in every future module must include `organization_id` — this becomes a
  checklist item in schema review from now on, not optional.
- No query currently filters by `organization_id`, since there's only one value — this is a known,
  accepted gap, not an oversight, and must be closed before this platform is ever used by more
  than one organization.
- This ADR should be revisited and superseded when actual multi-tenant routing is built, since the
  real tenant-isolation logic (row-level security, auth scoping) is a separate decision this ADR
  does not cover.
