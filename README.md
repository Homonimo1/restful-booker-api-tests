# Restful Booker API Regression Suite

An online Postman/Newman collection for the public Restful Booker sandbox. It exercises authentication, an owned booking's CRUD lifecycle, filtering, and negative authorization behavior.

## Run online

```bash
npm exec --yes --package=newman -- newman run collection.json --environment environment.json --reporters cli --timeout-request 15000 --delay-request 100
```

Run the collection in order. The public sandbox is shared and may be unavailable; this suite creates a run-specific booking and attempts to clean it up.

## What is asserted

The collection checks auth token handling, booking schema and payload values, full GET/PUT/PATCH values, persisted updates, that the unique-name filter includes this run's booking ID, and negative auth/status behavior against only this run's booking. Assertions are named as observable Then outcomes; request descriptions use Given/When/Then. PUT refreshes the expected full payload before sending; PATCH merges its payload into that expected state before sending.

## Live regression evidence

The integrated online collection was run on 2026-11-14: **21 requests, 36 assertions, 0 failures** (exit 0). It created, updated and deleted run-owned booking 3692; the post-delete GET and teardown GET both observed 404. The read-only constant invalid-ID request also returned 404. The pre-integration candidate separately passed 20 requests/35 assertions; the earlier remote README recorded a 2026-10-21 run with 20 requests/37 assertions and 0 failures. An earlier migration attempt ran 20 requests/37 assertions and failed four checks; it sent unauthenticated requests to existing ID 2, so it is not treated as successful evidence. That earlier run observed 500 for a create request missing firstname and 405 for PUT on a deleted ID; these are observations, not proven contract defects.

## Cleanup limits

Teardown runs only if Newman reaches it. An interrupted run or failed earlier request may leave the run-owned booking behind. The fallback teardown DELETE is not followed by a confirming GET. No offline harness or offline verification is included. All owned-booking requests, including unauthorized checks and the post-delete GET, use the run-created booking ID; a separate read-only invalid-ID request checks the constant ID 10000.

## Target and environment

`environment.json` supplies the public base URL; token, booking ID, unique run name and expected payload are runtime values. The collection uses the public sandbox demo credentials. Do not use these requests against production.
