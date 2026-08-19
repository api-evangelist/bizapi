---
name: Validate a BizAPI integration in the sandbox without spending credits
description: >-
  Exercise every BizAPI match method, plus the success and no-match paths, against the
  credit-free /cosearchtest endpoint using the provider's published fixture company, before
  pointing anything at the billable live endpoint.
api: openapi/bizapi-company-search-api-openapi.yml
operations:
  - searchCompaniesTest
generated: '2026-08-14'
method: generated
source: https://www.naics.com/wp-content/uploads/2021/09/BizAPI-V2-Documentation.pdf
---

# Validate a BizAPI integration in the sandbox

BizAPI charges per successful match out of a prepaid balance, so every call you make while
building should go to the sandbox. The sandbox is a separate endpoint, not a separate
credential.

## What the sandbox is

`searchCompaniesTest` is `POST /cosearchtest`. It takes the same body, requires the same HTTP
Basic credentials, and returns the same three-block envelope as the live operation — but the
appended data is fabricated and no credits are consumed.

There is **no test key prefix and no test credential**. The path is the only thing separating
test from live. Two consequences worth designing around:

1. A leaked sandbox credential is a live credential. Treat it as production secret material.
2. Nothing in the key itself tells your code, your linter or your secret scanner which mode it
   is in. Make the base path an explicit, reviewable config value rather than something derived
   at runtime.

## Step 1 — drive a successful append for each match method

The sandbox recognises one fixture company: **Westrock Mwv, LLC**. Use these bodies verbatim —
they are the provider's published values.

```json
{ "duns": "10-223-5004" }
```
```json
{ "companyName": "Westrock Mwv, LLC", "address": "501 S 5TH St", "city": "Richmond",
  "state": "VA", "postalCode": "23219", "country": "US", "phone": "8044441000" }
```
```json
{ "companyName": "Westrock Mwv, LLC", "address": "", "city": "", "state": "VA",
  "postalCode": "", "country": "US", "phone": "8044441000" }
```
```json
{ "companyName": "Westrock Mwv, LLC" }
```
```json
{ "url": "www.westrock.com" }
```
```json
{ "phone": "8044441000" }
```

Assert on the parsed envelope, not on the raw body: `Search Terms` echoes your input,
`Matching Data` carries the match metadata, `Appended Data` carries the record.

## Step 2 — drive the no-match path

This is the case most integrations get wrong, so test it explicitly. Put the string `bad` in
any input field:

```json
{ "duns": "bad" }
```
```json
{ "companyName": "bad", "address": "501 S 5TH St", "city": "Richmond", "state": "VA",
  "postalCode": "23219", "country": "US", "phone": "8044441000" }
```

Expected result: **HTTP 200** with

```json
"Appended Data": { "Message": "No match found" }
```

Your test must assert that this is treated as a miss. If your code reads only the status code,
this test is the one that catches it.

Note the substring rule: `bad` anywhere inside a value triggers the failure path. "Carlsbad
Market" is a no-match. If your fixtures include real companies with `bad` in the name, they
will fail in the sandbox and succeed in production — a false alarm to be aware of, not a bug.

## Step 3 — exercise the error paths you can reach

The sandbox shares the live error contract, so you can also verify:

- Empty body, or a body with none of `companyName`/`duns`/`url`/`phone` → 400
  "No valid search terms submitted…"
- Wrong or missing `Authorization` header → 401 "Credentials are Missing or Invalid."
- More than 3 requests in a rolling second → 429 "Too many requests. Please limit your requests
  to 3 per second"

You cannot simulate 403 credit exhaustion or the "Missing field: layout" 400 — both depend on
account state the sandbox does not expose. Handle them from the error catalog by inspection.

## Step 4 — verify your match-method exclusivity handling

Send a DUNS **plus** another field:

```json
{ "duns": "10-223-5004", "state": "VA" }
```

DUNS Match will not trigger, because "if any other fields are submitted with DUNS, URL, or
Phone Match, then these match methods will not trigger". Confirm your client builds
single-key bodies for the exclusive methods rather than merging everything it holds.

## Step 5 — cut over

Change only the path segment: `/cosearchtest` → `/cosearch`. Before you do, confirm you have
pacing at 3 requests per rolling second, that you log `Matching Data."Request ID"` on every
200, and that you have an alert on `Matching Data."Matches Remaining"` approaching zero.

## No-code alternative

The provider hosts a browser console at <https://www.naics.com/naics-api-test-V2/> — enter
credentials, fill the form, read the JSON. Useful for confirming what the account's Record
Layout actually returns before writing a parser.

The provider also publishes a Postman collection with all 18 of these calls prebuilt:
<https://www.naics.com/wp-content/uploads/2021/09/NAICS-BizAPI-V2-Examples.postman_collection.json_.zip>

## Related

- `sandbox/bizapi-sandbox.yml` — every published fixture value
- `errors/bizapi-problem-types.yml` — verbatim error strings to assert against
