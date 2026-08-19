---
name: Append firmographic data to a company record
description: >-
  Take a partial business record — company name, address, phone, website or DUNS number — and
  enrich it with NAICS/SIC codes, size figures and corporate linkage from BizAPI, choosing the
  right match method and handling the soft-failure and billing traps correctly.
api: openapi/bizapi-company-search-api-openapi.yml
operations:
  - searchCompanies
generated: '2026-08-14'
method: generated
source: https://www.naics.com/wp-content/uploads/2021/09/BizAPI-V2-Documentation.pdf
---

# Append firmographic data to a company record

BizAPI resolves one input record to at most one business establishment. Every successful match
costs a prepaid credit and cannot be undone or retried for free, so getting the match method
right on the first call is the whole job.

## Before you start

- Credentials are HTTP Basic, issued manually by NAICS Association. **The issued password
  contains spaces that are significant** — do not trim or collapse them before base64 encoding,
  or you will get a 401.
- Confirm which Record Layout the account is on. It is fixed at activation and decides which
  fields come back. You can read it from `Matching Data.Layout` on any response
  (NA, TA, EA, SA, PA, PL).
- If you are testing anything, use the `searchCompaniesTest` operation instead. Same
  credentials, no credits. See `bizapi-validate-integration-in-sandbox.md`.

## Step 1 — pick the match method from the data you hold

Send only the fields for the method you chose. This is not a preference, it is a hard rule:
**DUNS, URL and Phone match will not trigger if any other field is present in the request.**

| You hold | Method | Send exactly | Confidence |
|---|---|---|---|
| A DUNS number | DUNS Match | `duns` | 10 |
| Name + full address + phone | Standard Match | `companyName`, `address`, `city`, `state`, `postalCode`, `country`, `phone` | 7+ |
| Name + state only | Loose Match | `companyName`, `state` (others empty) | 8 |
| A website (US records only) | URL Match | `url` | 10 |
| Name + country | Name Match | `companyName`, `country` | 8 |
| A phone number | Phone Match | `phone` | 10 |

Clean the input first — it materially changes the match rate:

- `companyName` must contain exactly one company name. If you hold a second trading name, run
  a second call with it rather than concatenating.
- `address` holds the street address **or** a PO Box, never both. If a street match fails, retry
  with the PO Box.
- `phone` must be digits only. Strip separators and drop extensions.
- Keep fields under about 65 characters. Accented characters and symbols are not recognised and
  may become `?`.
- Never put a person's name, "ATTN TO:", a department or a second company into `address`.

## Step 2 — call searchCompanies

`searchCompanies` is `POST /cosearch` on `https://www.naics.com/wp-json/naicsapi/v2`.

Add any non-PII correlation field you need — an unrecognised key such as `Client#` is echoed
back in the `Search Terms` block so you can reattach the result to your row. Do not put PII in
these fields; the provider prohibits it.

## Step 3 — read the result correctly

The response is three blocks: `Search Terms`, `Matching Data`, `Appended Data`.

**Check for the miss first.** A no-match is **HTTP 200**, not a 404:

```
"Appended Data": { "Message": "No match found" }
```

If you branch on status codes alone you will record a phantom success on every unmatched row.

On a hit, gate on `Matching Data`:

- `Confidence Code` — 0-10. Anything the API returns is already 7 or higher; records at 6 or
  below are withheld entirely rather than flagged, so absence of a match may mean a weak match
  was suppressed.
- `Match Grade` — a seven-letter string, one letter per input element in order: company name,
  street number, street name, city, state, PO Box, telephone. `A` same, `B` similar, `F`
  different, `Z` null. DUNS, URL and Phone matches return no grade. A grade with `F` in the
  state or city position deserves a human look even at high confidence.
- `BEMFAB` — marketability. `M` marketable, `A` undeliverable, `D` de-listed at the company's
  request, `O` out of business or unconfirmed. **If `BEMFAB` is `D` you may append the industry
  code but you may not market to that company.** Carry this flag downstream; it is a
  contractual restriction inherited from D&B, not advice.
- `Matches Remaining` — your credit balance. Watch it. At zero the next call is a 403.
- `Request ID` — retain it. It is the only handle the provider can reconcile against, and it is
  not returned in a header, so you cannot recover it from a failed call.

## Step 4 — handle failures

| Status | Meaning | Do |
|---|---|---|
| 400 "No valid search terms submitted…" | You sent none of `companyName`, `duns`, `url`, `phone` | Fix the payload; do not retry as-is |
| 400 "Missing field: layout" | The account has no Record Layout configured | Stop. Contact the NAICS API contact — not client-fixable |
| 401 | Missing or invalid credentials | Check the password's internal spaces survived |
| 403 | Credits exhausted | Stop the run. Top up via APICloudSolutions@naics.com or 973-625-5626 |
| 429 | More than 3 requests in a rolling second | Back off. There is no `Retry-After` header — pace client-side |
| 500 | Server error | Retry once with backoff, then escalate |

## Rules that will bite you

- **There is no idempotency key.** A retried successful call is a second billable match. If a
  response is lost in transit you cannot tell a duplicate from a first attempt — log the
  `Request ID` from every 200 and reconcile rather than blindly retrying.
- **Pace at 3 requests per rolling second.** No rate-limit headers are returned at all. For a
  large backfill, ask APICloudSolutions@NAICS.com for a temporary boost rather than pushing
  through 429s.
- **Branch locations have blank size figures.** `Year Started`, `Employees Total` and
  `Sales Volume in US$` are only populated at headquarters. Check `Location Type` before
  treating a blank as "unknown" — for a branch it means "reported elsewhere".
- **There is no batch endpoint.** Large projects go through the provider manually.

## Related

- `conventions/bizapi-conventions.yml` — full request/response semantics
- `errors/bizapi-problem-types.yml` — verbatim error catalog and diagnostic field values
- `data-model/bizapi-data-model.yml` — entity and corporate-linkage graph
