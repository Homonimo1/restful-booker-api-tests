# Restful Booker - API Test Suite

API test suite for the public [Restful Booker](https://restful-booker.herokuapp.com) API, built with
**Postman** and runnable from the command line (and in CI) with **Newman**.

It covers authentication, the full booking CRUD lifecycle and negative cases.

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
| **AUTH** | `POST Token Auth` | Returns a token and stores it for the authenticated requests. |
| **Booking - Crud** | `GET Get_All_Books_Id` | The list of booking IDs is a non-empty array. |
| | `POST Created Booking` | Creates a booking, validates the response against a JSON schema and stores `booking_id`. |
| | `GET Get Boockin by ID` | Retrieves the created booking by ID. |
| | `PUT Update Boocking` | Full update (PUT) with the stored token. |
| | `PATCH Partial Update` | Partial update (PATCH): changes only `totalprice`. |
| **Negative case** | `POST CREATED BOOKING - Missing Fields` | Documents a server bug: returns `500` instead of `400`. |
| | `GET GET BOOKING - INVALID ID` | A non-existent ID returns `404`. |
| | `DELETE Delete Booking - Forbidden (no auth)` | Without a token the delete is rejected (`403`) and the booking still exists. |
| **Teardown** | `DELETE Delete Booking - Cleanup (auth)` | Removes the booking created during the run and verifies it is gone. |

## Environment variables

| Variable | Value | Description |
|----------|-------|-------------|
| `Url` | `https://restful-booker.herokuapp.com` | Base URL of the sandbox (shared value). |
| `token` | *(runtime)* | Filled automatically by `Token Auth`. |
| `booking_id` | *(runtime)* | Filled automatically by `Created Booking`. |

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
npm install -g newman
newman run collection.json -e environment.json
```

### Continuous integration

`.github/workflows/newman.yml` runs the suite with Newman on every push and pull request to
`main`/`master`, and can also be triggered manually from the **Actions** tab. The JSON report is
uploaded as a build artifact named `newman-report`.

## Coverage summary

- Authentication (success) and token chaining.
- Full booking lifecycle: create, read, update (PUT), partial update (PATCH) and delete.
- Response schema validation for the created booking.
- Negative cases: missing required fields, unknown ID, delete without authentication.
- Response body assertions on the returned data (not only status codes).

## Known issue

When required fields are missing from the create payload, the API replies **`500` instead of `400`**.
This is documented inside the collection as an explicit test rather than hidden, so the suite stays green
while the bug is visible.

## Note on the response-time assertion

`Get_All_Books_Id` asserts a response time **below 1000 ms**. The threshold is deliberately generous: the
suite runs against a shared public sandbox and also in CI, where network latency is higher than locally
(~100-400 ms locally). Keeps the check meaningful without making the pipeline flaky.

## Keeping the files in sync

After editing the collection in Postman, re-export it:

- **Collection** → `...` → *Export* → **Collection v2.1** → overwrite `collection.json`.
- **Environment** → `...` → *Export* → overwrite `environment.json`.

Remember to keep the environment with `token` and `booking_id` empty before exporting.
