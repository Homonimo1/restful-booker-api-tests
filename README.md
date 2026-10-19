# Restful Booker - API Test Suite

[![Newman API Tests](https://github.com/Homonimo1/restful-booker-api-tests/actions/workflows/newman.yml/badge.svg)](https://github.com/Homonimo1/restful-booker-api-tests/actions/workflows/newman.yml)

API test suite for the public [Restful Booker](https://restful-booker.herokuapp.com) API, built with
**Postman** and runnable from the command line (and in CI) with **Newman**.

It covers authentication, the full booking CRUD lifecycle and negative cases.

## Last verified run

| Metric | Value |
| --- | --- |
| Date | 2026-10-19 |
| Requests | 20 |
| Assertions | 38 |
| Failures | 0 |
| Duration | 4.2 s |

## Files

| File | Description |
|------|-------------|
| `collection.json` | Postman collection (requests, tests and descriptions). |
| `environment.json` | Environment with the base URL and the runtime variables. |
| `.github/workflows/newman.yml` | CI: runs the suite with Newman on every push and pull request. |

## API under test

- **Base URL:** `https://restful-booker.herokuapp.com`
- Public sandbox. Its documented credentials are `admin` / `password123` and are intentionally visible.

## Collection structure

| Folder | Request | What it validates |
|--------|---------|-------------------|
| **AUTH** | `POST token auth` | Returns a token and stores it for the authenticated requests. |
| | `POST token auth - bad credentials` | Wrong password: the API replies `200` with `"Bad credentials"` and no token (this API does not use `401`). |
| **Booking - CRUD** | `GET all booking IDs` | The list of booking IDs is a non-empty array. Stores an `existing_booking_id`. |
| | `POST create booking` | Creates a booking, validates the response against a JSON schema and stores `booking_id`. |
| | `GET booking by ID` | Retrieves the created booking by ID. |
| | `PUT update booking` | Full update (PUT) with the stored token. |
| | `PATCH partial update` | Partial update (PATCH): changes only `totalprice`. |
| | `GET verify update persisted` | Reads the booking again to prove the PUT/PATCH changes were **persisted on the server**, not just echoed in the response. |
| | `DELETE booking (authenticated)` | Closes the CRUD cycle: deletes the created booking and confirms it is gone (`404`). |
| **Filters** | `GET bookings by firstname` | Filters by `firstname`; an unknown name returns an empty array. |
| **Negative cases** | `POST create booking - missing fields` | Documents a server bug: returns `500` instead of `400`. |
| | `GET booking - invalid ID` | A non-existent ID returns `404`. |
| | `DELETE booking - no auth (403)` | Without a token the delete is rejected (`403`) and the booking is still there. Targets `existing_booking_id`, so it does not depend on the happy path. |
| | `PUT booking - invalid token (403)` | A tampered token is rejected (`403`) and the existing booking is still there. |
| | `PUT booking - non-existent ID (405)` | Documents the observed behavior: the API returns `405` instead of the expected `404`. |
| **Teardown** | `TEARDOWN cleanup booking` | Verifies the run left **no booking behind** (`404`). If a booking is still around (an earlier step failed) it is removed here. Clears the runtime variables. |

## Environment variables

| Variable | Value | Description |
|----------|-------|-------------|
| `Url` | `https://restful-booker.herokuapp.com` | Base URL of the sandbox (shared value). |
| `token` | *(runtime)* | Filled automatically by `POST token auth`. |
| `booking_id` | *(runtime)* | Filled automatically by `POST create booking`. |
| `existing_booking_id` | *(runtime)* | Filled by `GET all booking IDs`; used by the delete-forbidden case so it does not depend on the created booking. |

`token` and `booking_id` are runtime values: the environment ships with them empty and the collection fills
them on every run. The token is a credential and is never committed with a value.

## How to run

### Postman

1. Import `collection.json` and `environment.json`.
2. Select the **Restful Booker - QA** environment.
3. Open the **Collection Runner** and run the whole collection.

> The requests are **chained**: `{{token}}` and `{{booking_id}}` are produced by the first requests and
> consumed by the later ones, so the collection must be run **in order** (not request by request).

### Newman (command line / CI)

```bash
npm install -g newman@6
newman run collection.json -e environment.json
```

### Continuous integration

`.github/workflows/newman.yml` runs the suite with Newman on every push and pull request to
`main`/`master`, and can also be triggered manually from the **Actions** tab. Newman is pinned to
version 6 and the JSON report is uploaded as a build artifact named `newman-report`.

## Coverage summary

- Authentication (success), token chaining and a wrong-credentials negative case.
- Full booking lifecycle: create, read, update (PUT), partial update (PATCH) and delete. The happy path
  (`Booking - CRUD`) ends with the delete, so the folder really covers CRUD end to end.
- Persistence check: after PUT/PATCH the booking is read again to confirm the changes were stored server-side.
- Response schema validation for the created booking.
- Query filter by `firstname` (known name and unknown name).
- Negative cases: missing required fields, unknown ID, delete without authentication, tampered token and PUT on a non-existent ID.
- Response body assertions on the returned data (not only status codes).
- The negative delete targets an existing booking instead of the created one, so the negative folder does not
  depend on the happy path and the suite stays valid if requests are reordered.

## Known issue

When required fields are missing from the create payload, the API replies **`500` instead of `400`**.
This is documented inside the collection as an explicit test rather than hidden, so the suite stays green
while the bug is visible.

**PUT on a non-existent ID:** the API replies **`405` instead of `404`**. The suite asserts the observed
behavior and logs the response body, so the discrepancy stays visible.

## Note on the response-time assertion

`GET all booking IDs` asserts a response time **below 2000 ms**. The threshold is deliberately generous: the
suite runs against a shared public sandbox and also in CI, where network latency is higher than locally
(~100-400 ms locally). Keeps the check meaningful without making the pipeline flaky.

## License

ISC

## Keeping the files in sync

After editing the collection in Postman, re-export it:

- **Collection** → `...` → *Export* → **Collection v2.1** → overwrite `collection.json`.
- **Environment** → `...` → *Export* → overwrite `environment.json`.

Remember to keep the environment with `token` and `booking_id` empty before exporting.
