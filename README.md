# BizAPI (bizapi)
BizAPI is a real-time Business Intelligence API from the NAICS Association that provides firmographic data on over 220 million US and international business entities. It enables businesses to enrich CRM records, power customer acquisition workflows, and append NAICS codes, SIC codes, DUNS numbers, company details, sales volume, employee counts, and corporate hierarchy information to any business record via a simple REST API.

**URL:** [https://www.naics.com/business-intelligence-api/](https://www.naics.com/business-intelligence-api/)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - Business Intelligence, Company Data, CRM, Firmographic Data, NAICS, SIC

## Timestamps

- **Created:** 2025-02-24
- **Modified:** 2026-04-19

## APIs

### BizAPI Business Intelligence API
Real-time REST API from the NAICS Association that returns firmographic data on over 220 million US and international business locations. Provides DUNS numbers, SIC codes, NAICS codes, company details, contact information, sales volume, employee counts, and corporate hierarchy. Supports both live production and test (sandbox) endpoints. Rate limited to 3 requests per rolling second. Authentication via HTTP Basic with credentials provided at account activation.

**Human URL:** [https://www.naics.com/business-intelligence-api/](https://www.naics.com/business-intelligence-api/)

#### Tags:

 - Business Intelligence, Company Data, Firmographic Data, NAICS, SIC

#### Properties

- [Documentation](https://www.naics.com/business-intelligence-api/)
- [OpenAPI](openapi/bizapi-business-intelligence-api-openapi.yml)
- [JSONSchema](json-schema/bizapi-company-schema.json)
- [JSONStructure](json-structure/bizapi-company-structure.json)
- [JSON-LD](json-ld/bizapi-context.jsonld)
- [Example](examples/bizapi-company-example.json)

## Common Properties

- [Website](https://www.naics.com/)
- [Documentation](https://www.naics.com/business-intelligence-api/)
- [Sign Up](https://www.naics.com/bizapi-details/)
- [Authentication](https://www.naics.com/business-intelligence-api/)
- [SpectralRules](rules/bizapi-spectral-rules.yml)
- [NaftikoCapability](capabilities/bizapi-business-intelligence.yaml)
- [Vocabulary](vocabulary/bizapi-vocabulary.yaml)

## Features

| Name | Description |
|------|-------------|
| Real-Time Firmographic Data | Returns live firmographic data on over 220 million US and international business entities in real time. |
| NAICS and SIC Classification | Provides 6-digit NAICS codes and 4- and 8-digit SIC codes for industry classification of business entities. |
| DUNS Number Lookup | Returns D&B DUNS numbers enabling universal business entity identification and credit data linkage. |
| Corporate Hierarchy | Exposes parent, domestic ultimate, and global ultimate company relationships with DUNS and name fields. |
| CRM Enrichment | Designed to integrate with CRMs, SFAs, and internal systems to append firmographic data to business records. |
| Sandbox Test Endpoint | Includes a /cosearchtest endpoint that returns fake data without consuming API credits for development and testing. |

## Use Cases

| Name | Description |
|------|-------------|
| CRM Data Enrichment | Append NAICS codes, DUNS numbers, employee counts, and sales volume to company records in CRM and SFA systems. |
| Customer Acquisition | Identify and qualify business prospects by searching firmographic data to match against target industry and size criteria. |
| Market Research | Analyze business landscapes by querying firmographic data across industries, geographies, and corporate hierarchies. |
| Lead Scoring | Enrich inbound leads with firmographic attributes to power scoring models that prioritize high-value accounts. |
| Compliance Verification | Verify business identity, location, and corporate hierarchy for compliance and due diligence workflows. |

## Integrations

| Name | Description |
|------|-------------|
| Salesforce | Integrate BizAPI with Salesforce CRM to auto-append firmographic data to account and lead records. |
| HubSpot | Enrich HubSpot company records with NAICS, SIC, DUNS, and financial indicators via BizAPI. |
| Marketo | Append industry classification and company size data to Marketo lead records for segmentation and scoring. |
| Microsoft Dynamics | Connect BizAPI to Dynamics 365 to surface firmographic context on accounts and contacts. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [BizAPI Business Intelligence API](openapi/bizapi-business-intelligence-api-openapi.yml)

### JSON Schema

- [BizAPI Company](json-schema/bizapi-company-schema.json)

### JSON Structure

- [BizAPI Company](json-structure/bizapi-company-structure.json)

### JSON-LD

- [BizAPI Context](json-ld/bizapi-context.jsonld)

### Examples

- [BizAPI Company Example](examples/bizapi-company-example.json)

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [BizAPI Business Intelligence API](capabilities/shared/bizapi.yaml) — 2 operations for firmographic data lookup and enrichment

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [BizAPI Business Intelligence](capabilities/bizapi-business-intelligence.yaml) | BizAPI Business Intelligence API | 2 | Sales Representatives, Marketing Analysts, Data Engineers |

## Vocabulary

- [BizAPI Vocabulary](vocabulary/bizapi-vocabulary.yaml) — Unified taxonomy mapping 1 resource, 2 actions, 1 workflow, and 3 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [BizAPI Spectral Rules](rules/bizapi-spectral-rules.yml) — 30 rules across 13 categories enforcing BizAPI API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
