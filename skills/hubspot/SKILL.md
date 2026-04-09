---
name: hubspot
description: "Search and manage HubSpot CRM data — contacts, companies, and deals. Use when the user asks about customers, leads, deals, sales pipeline, or CRM data."
argument-hint: "<query or command>"
---

# HubSpot CRM

Search and manage contacts, companies, and deals in HubSpot using the v3 API.

## Authentication

The HubSpot access token must be available as `HUBSPOT_TOKEN` in the environment. If it's not set, check for a `.env` file in the project root and source it:

```bash
source .env 2>/dev/null
echo "${HUBSPOT_TOKEN:0:10}..."
```

If the token is not available, ask the user to set `HUBSPOT_TOKEN`.

## Interpreting Arguments

Parse `$ARGUMENTS` to determine what the user wants:

- **Search query** (e.g., `hubspot Marriott`) — search companies by default
- **Contact lookup** (e.g., `hubspot contact john@acme.com`) — get contact by email
- **Deal search** (e.g., `hubspot deals enterprise`) — search deals
- **Company details** (e.g., `hubspot company 12345`) — get company by ID
- **Contact search** (e.g., `hubspot contacts John Smith`) — search contacts
- **Deal details** (e.g., `hubspot deal 67890`) — get deal by ID

## API Reference

Base URL: `https://api.hubapi.com`
Auth header: `Authorization: Bearer $HUBSPOT_TOKEN`

### Search Companies

```bash
curl -s -X POST "https://api.hubapi.com/crm/v3/objects/companies/search" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $HUBSPOT_TOKEN" \
  -d '{
    "query": "SEARCH_TERM",
    "limit": 10,
    "properties": ["name", "domain", "industry", "numberofemployees", "annualrevenue", "hs_lead_status", "city", "country", "description", "notes_last_updated"]
  }'
```

### Search Contacts

```bash
curl -s -X POST "https://api.hubapi.com/crm/v3/objects/contacts/search" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $HUBSPOT_TOKEN" \
  -d '{
    "query": "SEARCH_TERM",
    "limit": 10,
    "properties": ["firstname", "lastname", "email", "company", "phone", "jobtitle", "hs_lead_status", "lifecyclestage", "notes_last_updated"]
  }'
```

### Get Contact by Email

```bash
curl -s -X POST "https://api.hubapi.com/crm/v3/objects/contacts/search" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $HUBSPOT_TOKEN" \
  -d '{
    "filterGroups": [{
      "filters": [{"propertyName": "email", "operator": "EQ", "value": "EMAIL_ADDRESS"}]
    }],
    "properties": ["firstname", "lastname", "email", "company", "phone", "jobtitle", "hs_lead_status", "lifecyclestage"]
  }'
```

### Search Deals

```bash
curl -s -X POST "https://api.hubapi.com/crm/v3/objects/deals/search" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $HUBSPOT_TOKEN" \
  -d '{
    "query": "SEARCH_TERM",
    "limit": 10,
    "properties": ["dealname", "amount", "dealstage", "closedate", "pipeline", "hubspot_owner_id", "notes_last_updated"]
  }'
```

### Get Object by ID

For any object type (companies, contacts, deals):

```bash
curl -s "https://api.hubapi.com/crm/v3/objects/OBJECT_TYPE/OBJECT_ID?properties=COMMA_SEPARATED_PROPS" \
  -H "Authorization: Bearer $HUBSPOT_TOKEN"
```

### Get Associated Objects

To find deals associated with a company, or contacts associated with a deal:

```bash
curl -s "https://api.hubapi.com/crm/v3/objects/OBJECT_TYPE/OBJECT_ID/associations/TARGET_TYPE" \
  -H "Authorization: Bearer $HUBSPOT_TOKEN"
```

Example — get deals for a company:
```bash
curl -s "https://api.hubapi.com/crm/v3/objects/companies/COMPANY_ID/associations/deals" \
  -H "Authorization: Bearer $HUBSPOT_TOKEN"
```

### Filter Operators

Available operators for `filterGroups`: `EQ`, `NEQ`, `LT`, `LTE`, `GT`, `GTE`, `CONTAINS_TOKEN`, `NOT_CONTAINS_TOKEN`, `HAS_PROPERTY`, `NOT_HAS_PROPERTY`, `IN`, `NOT_IN`.

## Workflow

1. **Parse the user's request** from `$ARGUMENTS`
2. **Run the appropriate API call** using `curl` and `jq`
3. **Format results** clearly — show key fields, skip nulls
4. **Follow up** if the user needs associations (e.g., "show me deals for this company"), fetch them
5. **Present concisely** — name, key fields, and IDs for reference

## Tips

- Use `jq` to parse and format JSON responses
- HubSpot search `query` does full-text search across default searchable properties
- For more precise filtering, use `filterGroups` instead of `query`
- Deal stages are IDs (not human-readable names) — fetch pipeline stages if the user needs readable names:
  ```bash
  curl -s "https://api.hubapi.com/crm/v3/pipelines/deals" \
    -H "Authorization: Bearer $HUBSPOT_TOKEN"
  ```
- Owner IDs map to HubSpot users — fetch owners if needed:
  ```bash
  curl -s "https://api.hubapi.com/crm/v3/owners" \
    -H "Authorization: Bearer $HUBSPOT_TOKEN"
  ```
- Maximum 10,000 results can be retrieved through search pagination
- Rate limit: 100 requests per 10 seconds for private apps
