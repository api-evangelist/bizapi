---
name: Resolve a company's corporate family tree
description: >-
  Walk from a single matched business establishment up to its parent, domestic ultimate and
  global ultimate using BizAPI's DUNS linkage fields, budgeting one billable call per hop and
  reading the hierarchy code correctly.
api: openapi/bizapi-company-search-api-openapi.yml
operations:
  - searchCompanies
generated: '2026-08-14'
method: generated
source: https://www.naics.com/wp-content/uploads/2021/09/BizAPI-V2-Documentation.pdf
---

# Resolve a company's corporate family tree

BizAPI returns corporate linkage as DUNS references, not as an expandable object. There is no
endpoint that returns a family tree. You walk it yourself, one `searchCompanies` call per hop,
and each hop is a billable match.

## Prerequisite

This only works on an account provisioned with the **Prospecting with Linkage (PL)** Record
Layout. Linkage fields are absent on NA, TA, EA, SA and PA. Check `Matching Data.Layout` on any
response before you build against this; if it is not `PL`, the fields below will simply not be
there and the answer is an account change, not a code change.

## Step 1 — get the anchor record

Call `searchCompanies` with your best available key (see
`bizapi-append-company-record.md` for choosing a match method). From `Appended Data`, read:

- `DUNS #` — the record you matched
- `Location Type` — `Headquarters`, `Branch`, or `Single Location`
- `Hierarchy Code` — two digits, the record's depth in the tree
- `Subsidiary Indicator` — whether this record is 51%+ owned by a parent
- `Global Ult Indicator` — `N` means this record is not itself the global ultimate
- `# Of Family Members` — the size of the whole tree

## Step 2 — read the linkage fields, minding the Location Type switch

The parent/HQ block means **two different things** depending on the anchor record:

- If the anchor is a **Headquarters or Single Location**, then `Parent Ult DUNS #`,
  `HQ/Parent Ult Bus. Name`, `HQ/Parent State/Province` and `HQ/Parent Country` describe the
  **parent company owning more than 50%**.
- If the anchor is a **Branch**, the same fields describe the **headquarters the branch reports
  to** — not an owner.

Branch it on `Location Type` before you label the relationship, or you will publish "parent
company" for what is actually a sibling location's HQ.

The two ultimate blocks are unambiguous:

- `Domestic Ult DUNS #` / `Domestic Ult Name` / `Domestic Ult State/Province` /
  `Domestic Ult Country` — the highest family member **in the same country** as the anchor.
- `Global Ult DUNS #` / `Global Ult Bus. Name` / `Global Ult State/Province` /
  `Global Ult Country` — the highest family member **regardless of country**.

A record can be its own domestic ultimate, its own global ultimate, or both. Compare the DUNS
values against the anchor's `DUNS #` before spending a call to look one up.

## Step 3 — interpret the hierarchy code

Two digits giving the record's position in the tree:

- Global ultimates are `01`.
- A subsidiary is one greater than its parent.
- A branch is equal to its headquarters.

So a record at `03` sits two levels below the global ultimate. Combined with
`# Of Family Members` — which counts the global ultimate, all subsidiaries and all their
branch locations — you can size the tree without walking it.

## Step 4 — hop, deliberately

To get size, classification or contact detail for any linked company, submit a **new**
`searchCompanies` request keyed on that DUNS alone:

```json
{ "duns": "08-109-2578" }
```

The provider states this directly: "To gain Company Size Details of the Global Ult, submit a
request based on the Global Ult DUNS #".

Budget before you loop:

- Every hop is one billable match. A three-hop walk on 10,000 anchors is 40,000 matches.
- Send the DUNS **alone**. Adding any other field suppresses DUNS Match entirely.
- Pace at 3 requests per rolling second, with no `Retry-After` to guide you.
- Deduplicate first. Many anchors in the same book of business share a global ultimate — resolve
  the distinct set of DUNS values, not one per row.
- Cache by DUNS. It is a stable identifier; re-resolving the same ultimate is pure waste.
- Watch `Matching Data."Matches Remaining"` on every response and stop the walk before it hits
  zero, or the run ends in a 403 partway through.

## Step 5 — stop conditions

Stop walking when any of these is true:

- `Global Ult Indicator` is not `N` — you are at the global ultimate.
- The linked DUNS equals the anchor's own `DUNS #` — self-reference.
- `Hierarchy Code` is `01`.
- The DUNS is already in your cache.

Guard the loop with a hop limit regardless. Linkage data is third-party and a cycle, however
unlikely, would be an expensive way to discover a data defect.

## Related

- `data-model/bizapi-data-model.yml` — the full relationship graph and field semantics
- `plans/bizapi-plans-pricing.yml` — which Record Layout carries linkage
