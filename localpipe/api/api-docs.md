<!--
Copy of https://www.localpipe.io/localpipe-api-docs.md fetched 2026-11-13.
Unmodified below this comment. The live URL is the source of truth; re-fetch it before relying on endpoint details.
-->

# LocalPipe API Documentation

## Authentication

All API requests require an API key passed in the `x-api-key` header.

```bash
curl -X POST https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/enrich \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"business_name": "Acme Corp", "business_website": "acme.com", "business_location": "Chicago, IL"}'
```

---

# ENDPOINT 1: Business Enrichment

## POST /enrich

Enriches a single business with owner/decision-maker contact information. All requests are **asynchronous** — the API returns a `202 Accepted` response immediately with a `job_id`. Results are delivered via webhook (`callback_url`) or can be polled.

**High-Volume Usage:** You can send thousands of requests simultaneously with a `callback_url` — every request is accepted immediately and queued. Results are delivered to your webhook as each enrichment completes. This is the recommended approach for **Clay** and other automation platforms.

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| business_name | string | Yes | The name of the business |
| business_website | string | No | The business website (with or without https://). If omitted, the system uses Google Search with the business name and location to find the owner. |
| business_location | string | No | City/state of the business (e.g. "Chicago, IL"). Improves accuracy for multi-location businesses. |
| business_phone | string | No | Recommended when requesting owner phones: supply the business listing phone as a string, preferably with its country code. Aliases: `phone`, `phone_number`. A valid mobile can be returned as `owner_phone` only with `want_phone=true` and an owner name; landlines and ambiguous results are not promoted. |
| want_name | boolean | No | Whether to find the owner/decision-maker name (default: true). Aliases: `want_owner_name`, `find_owner_name`. Set to `false` for business-email-only runs: name discovery is skipped, no name charge is applied, and `want_email` (owner email) is forced off since owner emails require a name. |
| want_email | boolean | No | Whether to find email (default: true, costs 2 credit) |
| want_phone | boolean | No | Whether to find phone (default: false, costs 4 credits) |
| want_general_email | boolean | No | Whether to find general business email (default: false). General emails are info@, contact@, hello@, etc. |
| general_email_fallback_only | boolean | No | Only find general business email if owner email is not found (default: false). Requires want_general_email to be true. |
| decision_maker_title | string | No | Target a specific role instead of the business owner (e.g. "Marketing Manager", "Practice Manager"). **If omitted, the system defaults to finding the business owner.** |
| owner_name | string | No | Supply the decision-maker's name yourself (e.g. "Jane Doe", at least 2 words). When provided, name discovery is skipped and contact sourcing uses this name directly. **Not billed** as a name-found credit and **never written to cache** (since it's unverified). Fallback business email logic still applies. |
| clay_row_id | string | No | A passthrough identifier returned unchanged in both the API response and webhook payload. Use this to match results back to rows in Clay or other automation platforms (e.g. `"{{_row_id}}"`). |
| callback_url | string | No | URL to receive results when ready (recommended for Clay and automation platforms). If omitted, poll for results using the job_id. |

```javascript
const response = await fetch("https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/enrich", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": "YOUR_API_KEY"
  },
  body: JSON.stringify({
    business_name: "Acme Corporation",
    business_website: "acme.com",
    business_location: "Chicago, IL",
    business_phone: "+1 202 555 0123", // optional business listing phone
    want_name: true,
    want_email: true,
    want_phone: true,
    want_general_email: true,
    general_email_fallback_only: false,
    decision_maker_title: "Marketing Manager",
    owner_name: "Jane Doe", // optional — skip name discovery and use this name
    clay_row_id: "row_abc123",
    callback_url: "https://your-webhook-url.com/callback"
  })
});
```

**Phone enrichment recommendation:** Pass `business_phone` from your Google Maps result or CRM and set `want_phone: true`. LocalPipe checks it with the existing mobile classification system. Supplying a phone does not enable phone enrichment by itself or prove that the number belongs personally to the owner. Only a valid mobile is eligible; landlines, VoIP, ambiguous results and failed checks are not promoted. Existing cache behavior still applies, and accepted owner-phone results use normal phone-find pricing.

#### Business-email-only example (skip name discovery)

Use this when you already have the owner's name or only need a general business email — no owner-name credit is charged and owner-email discovery is disabled automatically.

```javascript
const response = await fetch("https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/enrich", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": "YOUR_API_KEY"
  },
  body: JSON.stringify({
    business_name: "Acme Corporation",
    business_website: "acme.com",
    business_location: "Chicago, IL",
    want_name: false,
    want_email: false,
    want_general_email: true,
    clay_row_id: "row_abc123",
    callback_url: "https://your-webhook-url.com/callback"
  })
});
```

### Response (202 Accepted)

```json
{
  "success": true,
  "job_id": "abc-123",
  "total_records": 1,
  "message": "Request accepted. Record is queued. Results will be POSTed to your callback_url when ready.",
  "estimated_seconds": 30
}
```

### Response Field Reference

> ⚠️ **Field naming — read this before parsing responses.** The exact field names returned in webhook payloads, polling responses, and CSV exports are listed below. **Use these names verbatim in your parser.** A common mistake is looking for `general_email` (does not exist) — the general business email is always returned as `business_email`.

Every record object in `records[]` (webhook payload, polling response, and CSV row) contains exactly these fields:

| Field name (exact) | Type | Description | When populated |
|--------------------|------|-------------|----------------|
| `business_name` | string | Business name (echoed from your request) | Always |
| `business_website` | string | Website (echoed from your request) | Always |
| `business_location` | string | Location (echoed from your request) | Always |
| `owner_name` | string | Decision-maker full name, or `"not found"` | When `want_name=true` (default) |
| `owner_first_name` | string | First name split from `owner_name`, or `"not found"` | Same as `owner_name` |
| `owner_last_name` | string | Last name split from `owner_name`, or `"not found"` | Same as `owner_name` |
| `owner_email` | string | Verified personal email of the decision-maker, or `"not found"` | When `want_email=true` (default) |
| `owner_phone` | string | Decision-maker phone, or `"not found"` | When `want_phone=true` |
| `business_email` | string | **General business email** (`info@`, `contact@`, `hello@`, etc.), or `"not found"` | When `want_general_email=true` |
| `email_provider` | string \| null | Email provider classification for `owner_email` (e.g. `"google"`, `"microsoft"`) | When `owner_email` is found |
| `decision_maker_title` | string \| null | The title that was searched for this row | Multi-title jobs only |
| `owner_role` | string \| null | Role label associated with the decision-maker | When discovered |
| `extra_columns` | object \| null | Passthrough columns from your original request | When provided |

**Negative results are returned literally as the string `"not found"`** — never `null`, never an empty string, never missing. Your parser should treat `"not found"` as "no result for this field".

> **Field name pitfalls — DO NOT use these names, they don't exist:**
> - ❌ `general_email` → ✅ use `business_email`
> - ❌ `generic_email` / `info_email` → ✅ use `business_email`
> - ❌ `personal_email` → ✅ use `owner_email`
> - ❌ `first_name` / `last_name` → ✅ use `owner_first_name` / `owner_last_name`
> - ❌ `phone` / `mobile` → ✅ use `owner_phone`
> - ❌ `name` → ✅ use `owner_name` (and `business_name` for the company)

The request flag is `want_general_email`, but the **returned field is always `business_email`** — the names intentionally differ because the request asks "do you want me to look for it?" and the response key reflects what the value actually is (a generic business mailbox, not the owner's personal email).

### Enrichment Webhook Payload

```json
{
  "job_id": "abc-123",
  "status": "completed",
  "completed_at": "2025-01-01T12:00:00Z",
  "clay_row_id": "row_abc123",
  "stats": {
    "total_records": 1,
    "owner_names_found": 1,
    "owner_emails_found": 1,
    "owner_phones_found": 0,
    "general_emails_found": 1
  },
  "records": [
    {
      "business_name": "Acme Corp",
      "business_website": "acme.com",
      "business_location": "Chicago, IL",
      "owner_name": "John Smith",
      "owner_first_name": "John",
      "owner_last_name": "Smith",
      "owner_email": "john@acme.com",
      "owner_phone": "not found",
      "business_email": "info@acme.com",
      "email_provider": "google",
      "decision_maker_title": null,
      "owner_role": "Owner",
      "extra_columns": null
    }
  ]
}
```

> **Stats field naming:** the count of general business emails in the `stats` object is `general_emails_found` (matches the `want_general_email` request flag), while the per-record field is `business_email`. This is the only place in the response where "general" appears.

> **Email Verification Notice:** Both owner emails (`owner_email`) and business emails (`business_email`) are triple verified before they are returned, on every plan (credit-based and usage-based). Emails that fail verification come back as `"not found"`, so you do not need to run them through your own checker before sending.

### Parser checklist

Before shipping any merge/import script, confirm:
1. You read `business_email` (not `general_email`) for the generic mailbox.
2. You treat the literal string `"not found"` as an empty value — don't write it into your CRM.
3. You match webhook deliveries back to your rows using `clay_row_id` (the only field guaranteed to round-trip from your request).
4. You handle `owner_email` and `business_email` as separate columns — they are never merged on the API side.

### Enrichment Polling

```bash
curl "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/get-job-results?job_id=abc-123" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InV1bnJsZmZvbm52bWJzc3RrcHplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzA0OTIwMzMsImV4cCI6MjA4NjA2ODAzM30.8Ic_U17YrZtsDVnYnIP0XJzrhZJOLDSlJ0AZfTS31FQ"
```

---

## POST /cancel-enrich-job

Cancel one or more in-flight enrichment jobs, including jobs still `pending` (queued) that haven't started processing yet. Any records already enriched **remain preserved** and stay retrievable via `GET /get-job-results`. Only jobs still `pending` or `processing` are cancellable — jobs already `completed`, `failed`, or `cancelled` are returned in `already_terminal` untouched.

### Request

```bash
# Cancel one job
curl -X POST "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/cancel-enrich-job" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"job_id": "uuid-here"}'

# Cancel many at once (up to 500 per call)
curl -X POST "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/cancel-enrich-job" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"job_ids": ["uuid-a", "uuid-b", "uuid-c"]}'
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| job_id | string | Either this or `job_ids` | Single `job_id` returned by `/enrich`. |
| job_ids | string[] | Either this or `job_id` | Up to 500 `job_id`s to cancel in one call. |

### Response (200 OK)

```json
{
  "success": true,
  "cancelled": ["uuid-a", "uuid-b"],
  "already_terminal": [{ "job_id": "uuid-c", "status": "completed" }],
  "not_found": [],
  "failed": [],
  "message": "Cancelled 2 enrichment jobs. Records already enriched are preserved and remain retrievable via GET /get-job-results."
}
```

**Notes:**
- Queued jobs (`pending`, no records leased yet) cancel instantly with zero records processed.
- In-flight records already leased by workers finish out and are counted in the final stats; the job stops accepting new work immediately.
- Cancellation is safe to call twice on the same `job_id` — the second call returns it in `already_terminal` with `status: "cancelled"`.
- API keys can only cancel jobs owned by their user; unknown or foreign IDs come back in `not_found`.

---

# ENDPOINT 2: Google Maps Business Search

## POST /maps-search

Search Google Maps for businesses by keyword and location. Returns business names, phone numbers, addresses, ratings, websites, and more. All requests are **asynchronous** — returns a `task_id` immediately.

> ### ⚡ BATCH YOUR LOCATIONS — ALWAYS
>
> **Never loop cities.** `/maps-search` accepts an array of **up to 100 locations** in one call and fans them out in parallel under one `task_id`. This is MUCH faster, uses one rate-limit slot, and avoids dropped cities. Submitting one request per city is the #1 cause of incomplete data and rate-limit exhaustion.
>
> - ✅ **Correct:** `{ "query": "plumber", "location": ["Austin, TX","Dallas, TX","Houston, TX","San Antonio, TX"], "limit": 500 }`
> - ❌ **Wrong:** `for city in cities; do curl .../maps-search -d '{"query":"plumber","location":"'$city'"}'; done`
> - More than 100 locations? Split into the **minimum** number of batched requests (250 cities → 3 requests, not 250 requests of 1) and submit them through the `drain_queue` pattern in the Agent Mode section.

> ### ✅ BATCHING KEYWORDS IS SAFE
>
> `limit` is the total budget for the task, but it is **split evenly across every keyword × location pair**, so batching keywords no longer starves the later ones.
>
> | Your `limit` | What to do with keywords | Why |
> |---|---|---|
> | `"all"` / `null` (Find ALL) | **Batch freely** — up to 100 keywords in one `query` array | No budget to split; every keyword is exhausted independently |
> | Integer (e.g. `500`) | **Batch freely** — each pair gets `limit ÷ pairs` | Unused share from dry pairs is redistributed to pairs that still have matches |
> | Need a guaranteed N per keyword | Pass `limit = N × number_of_pairs` | Each pair's share then works out to N |
>
> With an integer `limit`, a batch of 5 keywords × 1 location and `limit: 500` gives each of the 5 pairs a starting share of 100, and any share a pair cannot use is redistributed to the others. A pair that comes back with `found: 0` is a real empty market, not truncation. Confirm with `sub_query_breakdown`.


### Sizing `limit`

`limit` is the ceiling for the whole task (max 50,000). To target N results per pair, set `limit = N × number_of_pairs` (pairs = keywords × locations) in **one** batched request:

| Shape | Ceiling |
|---|---|
| 1 request, `["plumber","roofer"]` × 1 city, `limit: 500` | 500 total, starting share 250 per keyword |
| 1 request, `["plumber","roofer"]` × 1 city, `limit: 1000` | 1,000 total, starting share 500 per keyword |

Each pair is capped at its even share first; if a pair has fewer matches than its share, the unused part goes to pairs that still have more. A pair only ends up with fewer than its share when it genuinely has fewer matches.


**Exact Category Matching:** Your `query` is matched against Google Maps' official business category taxonomy (e.g. "Plumber", "Dentist", "HVAC Contractor"). When a match is found, results are filtered to only include businesses with that exact category — eliminating irrelevant results. Each result includes a `matched_category` field showing which category it matched. If no exact category match is found, results are returned based on keyword relevance. **Custom keywords are still possible** — queries that don't match Google's taxonomy are accepted and run as relevance searches, but they will not populate `matched_category`.

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| query | string or string[] | Yes | Business type/keyword(s). **Pass as an array to batch up to 100 keywords in ONE request** instead of looping — e.g. `["plumbers","hvac contractor","electrician"]`. Matched against Google Maps category taxonomy for exact filtering. |
| location | string or string[] | Yes | City names, state names, country names, US ZIP codes, or UK postcodes. **Pass as an array to batch up to 100 locations in ONE request** instead of looping — e.g. `["Austin, TX","Dallas, TX","Houston, TX"]`. Do NOT submit one request per city. |
| limit | integer \| "all" \| null | No | Total budget for the task. Default: 100, minimum: 50, max: 50,000. Values below 50 are raised to 50. ⚠️ **With multiple keywords and/or locations the budget is split evenly across every (query × location) pair**, and any share a dry pair cannot use is redistributed to pairs that still have more matches. 2 queries × 4 locations with `limit: 400` → 8 pairs, 50 each, and unused shares flow to the pairs with more results. **Pass `"all"` (or `null`) for Find ALL mode** — no budget at all, every pair returns everything Google has. |
| filters | object | No | Filter results (see Filter Options below) |
| callback_url | string | No | Webhook URL for result delivery when search completes |
| exclude_existing_lists | string[] | No | **Exclude already found.** Business List IDs to compare against. Any business already in those lists (matched by `place_id` or website domain) is removed from this task's results. Get the IDs from `GET /list-business-lists`. Max 50 lists per request. IDs that don't belong to your account return `400`. |

### Exclude Already Found

Pass `exclude_existing_lists` to get only businesses you do not already have. The exclusion is applied to the results of the task, so a `limit` of 500 can return fewer than 500 rows when many matches were already in your lists. Ask for more than you need if you require a guaranteed count.

```javascript
// 1. Look up your lists
const lists = await fetch("https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/list-business-lists", {
  headers: { "x-api-key": "YOUR_API_KEY" }
}).then(r => r.json());

// 2. Search, excluding everything already in those lists
const res = await fetch("https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/maps-search", {
  method: "POST",
  headers: { "Content-Type": "application/json", "x-api-key": "YOUR_API_KEY" },
  body: JSON.stringify({
    query: "plumbers",
    location: ["Austin, TX", "Dallas, TX"],
    limit: 1000,
    exclude_existing_lists: lists.lists.map(l => l.list_id)
  })
});
```

`GET /get-scrape-results` then reports how many were skipped:

```json
{
  "exclude_existing": {
    "lists_compared": 3,
    "excluded_from_results": 212,
    "note": "Businesses already present in the Business Lists you passed to /maps-search were removed from these results."
  }
}
```

## GET /get-usage (account type, credits and Google Maps exports)

Returns your account type, remaining credits and remaining Google Maps exports for the current billing cycle. Use it to check what you have left before submitting large jobs.

```bash
curl "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/get-usage" \
  -H "x-api-key: YOUR_API_KEY"
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| start_date | ISO date | current cycle start | Start of the usage window |
| end_date | ISO date | now | End of the usage window |

### Response (credit plan accounts)

```json
{
  "account_type": "credit_plan",
  "billing_mode": "credits",
  "plan": "growth",
  "period_start": "2026-10-01T00:00:00.000Z",
  "period_end": "2026-10-23T20:10:00.000Z",
  "credits_remaining": 7420,
  "credits_monthly_allowance": 10000,
  "business_exports_limit": 20000,
  "business_exports_used": 3150,
  "business_exports_remaining": 16850,
  "credits_used": 2580,
  "records_processed": 1900
}
```

| Field | Description |
|-------|-------------|
| account_type | `credit_plan` or `usage_billing`. Check this first. |
| credits_remaining | Credits left to spend on enrichment finds |
| credits_monthly_allowance | Credits granted per billing cycle |
| business_exports_remaining | Google Maps exports left for Google Maps business search results |
| business_exports_limit | Total Google Maps exports this cycle (including rollover) |

### Usage billing accounts

If `account_type` is `usage_billing`, you are billed in dollars from your prepaid balance, not in credits. **Ignore any credit or Google Maps export counts** for these accounts; they do not limit your usage. Use `usage_balance_cents` (your remaining balance) and `total_usage_spent_cents` instead.

```json
{
  "account_type": "usage_billing",
  "billing_mode": "usage",
  "plan": "usage",
  "usage_balance_cents": 4250,
  "total_usage_spent_cents": 18730
}
```

This endpoint is read-only and costs no credits.

## GET /list-business-lists

Lists your saved Business Lists so you can see what you already have and feed IDs into `exclude_existing_lists`.

```bash
curl "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/list-business-lists?limit=100" \
  -H "x-api-key: YOUR_API_KEY"
```

### Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| limit | integer | 100 | Lists per page (1-200) |
| offset | integer | 0 | Pagination offset |
| search | string | — | Case-insensitive match on the list name |

### Response

```json
{
  "total_lists": 12,
  "lists": [
    {
      "list_id": "0f1c8a2e-4d3b-4f77-9b2a-1c5f9d8e7a10",
      "name": "Austin Plumbers",
      "keyword": "plumbers",
      "locations": ["Austin, TX"],
      "total_leads": 1843,
      "enrichment_status": "completed",
      "created_at": "2026-09-01T12:04:11.000Z",
      "updated_at": "2026-09-01T12:41:02.000Z"
    }
  ],
  "pagination": { "limit": 100, "offset": 0, "next_offset": null, "has_more": false, "returned": 12 }
}
```

This endpoint is read-only and costs no credits. It counts toward your per-minute request limit.

## /business-list-api (actions on an existing Business List)

One endpoint for everything you can do to a saved list. Every call needs `list_id` (from `GET /list-business-lists`) and your `x-api-key`. Lists that don't belong to your account return `404`. POST calls choose what to do with `action`.

| Action | Method | What it does | Cost |
|--------|--------|--------------|------|
| (none) | GET | List summary: name, total_leads, chain-clearing and review-scrape status | Free |
| `companies` | GET | Paginated companies with company IDs for editing, moving and removal | Free |
| `merge` | POST | Copy all companies from source lists into a destination, optionally moving them | Free |
| `move_companies` | POST | Move selected companies from one source into a destination list | Free |
| `edit_company` | POST | Update fields on one company in a list | Free |
| `remove_companies` | POST | Permanently remove selected companies from a list | Free |
| `enrich` | POST | Sends the list's businesses through enrichment as one job | Same as the enrichment you request (billed per result found, like `/enrich`) |
| `delete` | POST | Permanently deletes the list and its businesses | Free |
| `clean_chains` | POST | Finds (`mode: "preview"`) or removes (`mode: "apply"`) chain / franchise businesses | Free |
| `scrape_reviews` | POST | Starts a background Google review scrape for every business in the list | Free |
| `cancel_reviews` | POST | Stops a running review scrape and clears partial reviews | Free |
| `reviews` | GET | Review scrape progress and the scraped reviews, paginated by business | Free |

Base URL: `https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/business-list-api`

### Read companies and their IDs

`GET /business-list-api?list_id=LIST_ID&action=companies&limit=100&offset=0`

Returns `{ "list_id": "...", "total_companies": 200, "companies": [{ "company_id": "UUID", "place_id": "...", "position": 0, "fields": { "name": "Example", "website": "https://example.com" } }], "pagination": { "limit": 100, "offset": 0, "next_offset": 100 } }`.
Use `company_id` (not Google Place ID) for the actions below. Limit 1-1,000; offset is a non-negative integer. A company copied or moved into another list receives a new company ID; read the destination again. Pagination ends when `next_offset` is null.

### Merge lists into one larger list

`list_id` is an existing destination list; provide 1-50 source list IDs. Sources remain intact by default.

`{ "action": "merge", "list_id": "DESTINATION_UUID", "source_list_ids": ["SOURCE_UUID_1", "SOURCE_UUID_2"], "move": false }`

Set `move: true` to remove source companies after merging. Source lists remain as empty lists, and can be deleted separately. Duplicate Google Place IDs are included once; existing destination details win. Records without a Place ID remain separate. No new search, enrichment, credit charge or refund takes place.

### Move selected companies

`{ "action": "move_companies", "list_id": "DESTINATION_UUID", "source_list_id": "SOURCE_UUID", "company_ids": ["COMPANY_UUID"] }`

Moves 1-1,000 selected companies out of the source list. Companies already present by Place ID are removed from the source without duplicating them in the destination. Original company fields are preserved; destination details win for duplicates.

### Edit a company

`{ "action": "edit_company", "list_id": "LIST_UUID", "company_id": "COMPANY_UUID", "fields": { "name": "Updated business", "website": "https://example.com", "phone_number": "+12125551234" } }`

Editable fields: `name`, `website`, `phone_number`, `full_address`, `city`, `price_level`, `rating` (0-5), `review_count` (non-negative integer), `types` (string array), `latitude` (-90 to 90), `longitude` (-180 to 180). Omitted fields are unchanged; null clears a field except name, which cannot be blank. Company ID, Place ID and ownership cannot be edited. These edits affect the saved list only, not completed enrichment results or Google's listing.

### Remove companies from a list

`{ "action": "remove_companies", "list_id": "LIST_UUID", "company_ids": ["COMPANY_UUID_1", "COMPANY_UUID_2"] }`

Permanently removes 1-1,000 selected companies without deleting the list. List totals update immediately. No credits are refunded.

All merge/move/edit/remove changes are atomic: either the entire request succeeds or nothing changes. Every affected list must belong to your API-key account. Missing lists or companies return `404`; invalid inputs return `400`; lists with an active scrape, review scrape, location scrape or list-view preparation return `409` (retry after completion). These actions do not cancel background work or rewrite existing enriched lists.

Success returns `{ "success": true, "action": "merge", "list_id": "...", "selected_count": 200, "added_count": 180, "removed_count": 0, "duplicate_count": 20, "updated_count": 0, "list_totals": { "DESTINATION_UUID": 500, "SOURCE_UUID": 200 } }`. Counts reflect actual changes. `removed_count` includes source duplicates removed by a move; `updated_count` is 1 for an edit.

### Enrich a list

```bash
curl -X POST "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/business-list-api" \
  -H "x-api-key: YOUR_API_KEY" -H "Content-Type: application/json" \
  -d '{"action":"enrich","list_id":"LIST_ID","want_email":true,"want_business_email":true,"callback_url":"https://your-server.com/webhook"}'
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| want_name | boolean | No (default true) | Find owner names |
| want_email | boolean | No (default true) | Find owner emails (needs want_name) |
| want_phone | boolean | No (default false) | Find owner mobiles (needs want_name) |
| want_business_email | boolean | No (default false) | Find business emails |
| business_email_fallback_only | boolean | No (default false) | Only look for a business email when no owner email is found |
| decision_maker_title | string | No | Target a title instead of the owner, e.g. "Office Manager" |
| count | integer | No | Only enrich the first N businesses (max 100,000) |
| only_with_website | boolean | No (default false) | Skip businesses with no website |
| callback_url | string | No | Webhook for results, same payload as `/enrich` jobs. If omitted, poll instead |

Response `202`: `{ "success": true, "job_id": "...", "total_records": 1843, "status": "pending" }`. Track it with `GET /get-job-status?job_id=...` and fetch rows with `GET /get-job-results?job_id=...`. Each row's `extra_columns` carries the business's Google Place ID, phone and address so you can join results back to the list. Balance and plan rules are the same as `/enrich` (`402` when you can't run the job; a `queued_hold` object when it is accepted but waiting for a top-up).

### Delete a list

```json
{ "action": "delete", "list_id": "LIST_ID" }
```
Response: `{ "success": true, "list_id": "...", "deleted": true }`. This cannot be undone.
Deletion works through the API as well as the dashboard. It removes the saved list and its companies, prevents its source scrape from recreating it, and does not delete existing enriched lists or refund credits.

### Clear chains

```json
{ "action": "clean_chains", "list_id": "LIST_ID", "mode": "preview" }
```
`preview` (default) changes nothing and tells you what would be removed; `apply` removes them. Response: `{ "total_leads", "chains_found", "removed_count", "remaining_count", "by_brand": [{ "brand", "category", "count", "examples" }] }`. `removed_count` is always 0 in preview.

### Scrape Google reviews for a list

```json
{ "action": "scrape_reviews", "list_id": "LIST_ID", "max_per_company": 20, "rating_min": 1, "rating_max": 5 }
```
`max_per_company` 1-100 (default 20); `rating_min` / `rating_max` 1-5. Returns `202` and runs in the background. Starting again while one is running resumes it. Poll:

```bash
curl "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/business-list-api?list_id=LIST_ID&action=reviews&limit=100&offset=0" \
  -H "x-api-key: YOUR_API_KEY"
```

`review_scrape.status` is `not_started`, `processing`, `completed`, `failed` or `canceled`, with `progress` 0-100. `businesses` is `[{ "place_id", "reviews": [...] }]`, up to 500 per page (`pagination.next_offset` is null on the last page). Businesses with no matching reviews are not listed. For a single business, use `POST /scrape-reviews-api` instead.


### Filter Options

| Filter | Type | Description |
|--------|------|-------------|
| min_rating | number | Minimum Google rating (1-5) |
| max_rating | number | Maximum Google rating (1-5) |
| min_reviews | number | Minimum review count |
| max_reviews | number | Maximum review count |
| has_website | boolean | `true` returns only businesses **with** a website. `false` returns only businesses **without** a website (useful for web-design prospecting). Omit the field to allow both. |
| has_phone | boolean | `true` returns only businesses **with** a phone number. **Do not pass `false`** — it would exclude every listing that has a phone, which collapses yields (especially outside the US). Omit the field to allow both. |
| price_level | string[] | Google price ranges: ["$1-10", "$10-20", "$20-30", "$30-50", "$50-100", "$100+"]. Select one or more. Legacy $, $$, $$$, $$$$ remain supported; businesses without price data stay included. Most useful for restaurants/Hospitality. |
| opens_before | number | Business opens before this hour (0-23) |
| closes_after | number | Business closes after this hour (0-23) |
| radius_km | number | Search radius in kilometers around each ZIP/postcode location (3-80, default 25; values below 3 are raised to 3). Searches cover the radius around the ZIP/postcode center, not exact postal boundaries. Only applies to ZIP/postcode locations — ignored for city/state/country names. |

### Example Request

```javascript
const response = await fetch("https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/maps-search", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": "YOUR_API_KEY"
  },
  body: JSON.stringify({
    query: "plumbers",
    location: ["Austin, TX", "San Antonio, TX"],
    limit: 500,
    filters: {
      min_rating: 4.0,
      has_website: true,
      has_phone: true
    },
    callback_url: "https://your-webhook-url.com/maps-callback"
  })
});
```

### Multi-Keyword + Multi-Location Batch Example (Recommended)

This is the API's core strength: pass **all your keywords** and **all your cities** in a single call. Server-side fan-out runs every (keyword × location) pair in parallel and returns everything under one `task_id`.

```javascript
const response = await fetch("https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/maps-search", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "x-api-key": "YOUR_API_KEY"
  },
  body: JSON.stringify({
    query: ["plumbers", "hvac contractors", "electricians", "roofers", "painters"],  // up to 100 keywords
    location: [
      "Austin, TX", "Dallas, TX", "Houston, TX", "San Antonio, TX", "Fort Worth, TX",
      "El Paso, TX", "Arlington, TX", "Corpus Christi, TX", "Plano, TX", "Lubbock, TX"
    ],  // up to 100 locations
    limit: 300,  // total budget, split evenly across all 50 pairs (6 each), unused share redistributed
    filters: {
      has_website: true,
      min_rating: 3.5
    },
    callback_url: "https://your-webhook-url.com/maps-callback"
  })
});
```

**What this single call does:**
- Fans out to **5 keywords × 10 locations = 50 parallel searches** server-side
- The **whole task** returns up to **300 leads total**, split evenly across the 50 pairs (6 each), with unused share from dry pairs redistributed to the rest
- Consumes **exactly 1 rate-limit slot** on submission (not 50)
- Want thousands of leads? Raise `limit`, or use `"limit": "all"` for full coverage

> ⚠️ **If you need more than 100 keywords or 100 locations:** split into the **minimum** number of batched requests. 250 cities → 3 requests (100, 100, 50 locations, all keywords in each). Never send one request per city or per keyword.

### Find ALL Mode vs. an integer limit

`limit` is the maximum number of results **for the whole task**, but it is shared out **per (query × location) pair** rather than first-come-first-served.

- **Integer limit** — the budget is divided evenly across pairs. For example, `query: ["plumber"], location: ["Austin, TX", "Dallas, TX"], limit: 300` gives each city a share of 150. If Austin only has 40 matches, the leftover 110 goes to Dallas, so the task still returns up to 300 in total. Want a guaranteed 300 per city? Pass `limit: 600` for two cities.
- **Find ALL (`"limit": "all"` or `null`)** — use this when you want **complete coverage** of every available lead in your areas. There is no budget at all; the scrape keeps pulling until Google stops returning new businesses for each pair.


Use Find ALL when the goal is "every business I can get in these locations." Use an integer when the goal is "give me up to N leads from this request in total."

```json
{
  "query": ["plumber", "electrician", "hvac contractor", "roofer", "painter"],
  "location": ["Tweed Heads, NSW", "Coolangatta, QLD", "Robina, QLD", "Burleigh Heads, QLD"],
  "limit": "all",
  "callback_url": "https://your-webhook-url.com/maps-callback"
}
```

**When to use Find ALL:**
- ✅ You want systematic coverage of a region (every plumber in these 20 postcodes)
- ✅ You want a multi-vertical industry sweep across a fixed footprint
- ✅ You'd rather receive "everything Google knows" than guess a per-area cap
- ❌ Skip it when you truly only want a top-N sample — pass an integer instead

Combined with multi-keyword × multi-location batching (up to 100 keywords × 100 locations under one `task_id`), a single Find-ALL request can return tens of thousands of leads.

> ### 🎯 How `limit` is applied
>
> `limit` is the total budget for the task, and for multi keyword × location jobs that budget is **divided evenly across every pair** before results are returned.
>
> Example: `query: ["plumber","electrician"]` + `location: ["Austin, TX","Dallas, TX"]` + `limit: 300` creates 4 pairs, so each pair starts with a budget of 75. If a pair has fewer matches than its share (or zero), the leftover is **redistributed** to the pairs that are still truncated, repeatedly, until the budget is spent or every pair is fully served. No pair can consume the whole task and starve the others, and no budget is wasted on dry pairs.
>
> `limit_scope` tells you how the budget was applied: `"per_keyword_location_pair"` when there is more than one pair (still one total budget, split evenly), `"task"` for a single keyword + single location, and `"all"` for Find ALL. The results response (`GET /get-scrape-results`) also includes `per_pair_budget` (each pair's starting even share, `floor(limit ÷ pairs)`) for multi-pair tasks with an integer limit; the 202 submit response does not.
>
> Want N leads per city or per keyword? Pass `limit = N × number_of_pairs`, or use `"limit": "all"` for exhaustive coverage.
>
> After the job completes, read `sub_query_breakdown` in the response to see how many leads were found vs. returned for **each location and each keyword**. `found: 0` means that pair genuinely had no matches. This field is returned only for multi keyword × location tasks — single keyword + single location tasks omit it.



### Response (202 Accepted)

```json
{
  "success": true,
  "task_id": "uuid-here",
  "status": "pending",
  "query": ["plumbers"],
  "locations": [
    { "input": "Austin, TX", "resolved": "Austin, US" },
    { "input": "San Antonio, TX", "resolved": "San Antonio, US" }
  ],
  "limit": 500,
  "limit_scope": "per_keyword_location_pair",
  "message": "Search queued. Poll GET /get-scrape-results?task_id=uuid-here for results."
}
```

## Retrieving Maps Search Results

### Option 1: Webhook (Recommended)

Include a `callback_url` in your request. When the search completes, results are POSTed to your URL:

```json
{
  "task_id": "uuid-here",
  "status": "success",
  "total_results": 476,
  "query": ["plumbers"],
  "locations": ["Austin, TX", "San Antonio, TX"],
  "data": [
    {
      "name": "Austin Plumbing Co",
      "phone": "+15125551234",
      "full_address": "123 Main St, Austin, TX 78701",
      "city": "Austin",
      "rating": 4.8,
      "review_count": 342,
      "website": "https://austinplumbing.com",
      "types": ["plumber", "contractor"],
      "matched_category": "Plumber",
      "price_level": "$20-30",
      "place_id": "ChIJ...",
      "business_id": "0x...",
      "latitude": 30.2672,
      "longitude": -97.7431,
      "is_claimed": true,
      "working_hours": { "Monday": "8AM-6PM", "Tuesday": "8AM-6PM" }
    }
  ]
}
```

### Option 2: Polling (with Pagination)

Poll for results using the `task_id`. Large result sets are paginated to stay within the 6 MB response limit.

```bash
# Check progress (or fetch first page if complete)
curl "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/get-scrape-results?task_id=uuid-here" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InV1bnJsZmZvbm52bWJzc3RrcHplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzA0OTIwMzMsImV4cCI6MjA4NjA2ODAzM30.8Ic_U17YrZtsDVnYnIP0XJzrhZJOLDSlJ0AZfTS31FQ"

# Fetch a specific page
curl "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/get-scrape-results?task_id=uuid-here&limit=5000&offset=5000" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InV1bnJsZmZvbm52bWJzc3RrcHplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzA0OTIwMzMsImV4cCI6MjA4NjA2ODAzM30.8Ic_U17YrZtsDVnYnIP0XJzrhZJOLDSlJ0AZfTS31FQ"
```

### Query Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| task_id | string | — | **Required.** The task ID returned by /maps-search. |
| limit | integer | 5000 | Page size. Max **5000**. |
| offset | integer | 0 | Number of leads to skip. |
| relevance_filter | string | off | Filter out off-topic results: `off` (default), `moderate`, or `strict`. Uses keyword stems against business types and names — useful when scraping broad terms like "consulting" that pull in unrelated categories. |

**Processing response:**
```json
{
  "task_id": "uuid-here",
  "status": "processing",
  "progress": {
    "completed_work_units": 15,
    "total_work_units": 45,
    "leads_found": 230,
    "target_limit": 500
  }
}
```

**Completed response (paginated):**
```json
{
  "task_id": "uuid-here",
  "status": "success",
  "total_results": 12450,
  "query": ["plumbers"],
  "locations": ["Texas"],
  "pagination": {
    "limit": 5000,
    "offset": 0,
    "next_offset": 5000,
    "has_more": true,
    "returned": 5000,
    "warning": "MORE LEADS AVAILABLE. Do NOT submit /enrich yet — fetch all pages first by re-calling this endpoint with offset=5000 until has_more=false."
  },
  "enrichment_contract": {
    "total_leads_to_enrich": 12450,
    "required_enrich_submissions": 12450,
    "instruction": "This scrape returned 12450 leads. You MUST submit exactly 12450 POST /enrich requests (one per lead) to enrich them all. Submitting fewer is a data loss bug — assert your enrich submission count == 12450 before declaring the workflow complete."
  },
  "data": [ ... ]
}
```

### Per-sub-query breakdown (batched jobs)

When a task was submitted with multiple keywords and/or locations **and** an integer `limit`, the response also includes:

Example: `query: ["dermatology clinic"]`, `location: ["Sydney, Australia", "Perth, Australia"]`, `limit: 300`. Each city starts with 150; Perth has no matches, so its whole share moves to Sydney, which returns the full 300.

```json
{
  "limit_scope": "per_keyword_location_pair",
  "per_pair_budget": 150,
  "sub_query_breakdown": {
    "by_location": [
      { "location": "Sydney, Australia", "found": 505, "returned": 300, "truncated": true },
      { "location": "Perth, Australia", "found": 0, "returned": 0, "truncated": false }
    ],
    "by_keyword": [
      { "keyword": "dermatology clinic", "found": 505, "returned": 300, "truncated": true }
    ]
  }
}
```

`limit` is one total budget for the job, split evenly across pairs (`per_pair_budget`), with any share a pair cannot use redistributed to pairs that still have more matches. Use `sub_query_breakdown` to tell **"this sub-query was truncated"** (`truncated: true` — raise `limit` or use Find ALL) apart from **"this sub-query genuinely had no matches"** (`found: 0`). You never need to split a batch into one call per keyword/city.


> **🚨 AI AGENTS / AUTOMATIONS — READ THIS.** The `enrichment_contract.required_enrich_submissions` field is the **ground-truth count** of `/enrich` calls you must submit. We see automations regularly scrape 5,000–20,000 leads then submit only 400–1,000 enrich requests because they processed one page and stopped. That is a silent data-loss bug and the user is paying for leads they never receive. Before declaring the workflow done: (1) paginate `/get-scrape-results` until `has_more=false`, accumulating every lead; (2) count what you accumulated and assert it equals `total_results`; (3) submit one `/enrich` per lead and assert your submission count equals `enrichment_contract.required_enrich_submissions`. If you exit early because of a shell timeout, resume from the last `offset` — do not skip leads.

**Pagination loop example:**
```javascript
let offset = 0;
const allLeads = [];
while (true) {
  const res = await fetch(
    `https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/get-scrape-results?task_id=${taskId}&limit=5000&offset=${offset}`,
    { headers: { apikey: ANON_KEY } }
  );
  const page = await res.json();
  if (page.status !== "success") break; // still processing
  allLeads.push(...page.data);
  if (!page.pagination?.has_more) break;
  offset = page.pagination.next_offset;
}
```

> **Note on webhooks:** Webhook payloads always include the **full result set** in a single POST (no pagination). For very large jobs (>10,000 leads), prefer polling with pagination to avoid exceeding your endpoint's payload limit.

### Business Data Fields

Each business in the `data` array contains:

| Field | Type | Description |
|-------|------|-------------|
| name | string | Business name |
| phone | string | Phone number |
| full_address | string | Full street address |
| city | string | City name |
| rating | number | Google rating (1-5) |
| review_count | number | Number of Google reviews |
| website | string | Business website URL |
| types | string[] | Google business categories |
| matched_category | string | The exact category from Google's taxonomy that this business matched (null if no exact match) |
| price_level | string or null | Google's reported price range (e.g. $20-30 or $100+), legacy dollar marks, or null when unavailable |
| place_id | string | Google Maps Place ID |
| business_id | string | Google internal business ID (CID) |
| latitude | number | Latitude |
| longitude | number | Longitude |
| is_claimed | boolean | Whether the business has claimed its Google listing |
| working_hours | object | Operating hours by day |

---

## POST /cancel-scrape-job

Cancel one or more in-flight Google Maps scrape jobs. Any leads already scraped **remain preserved** in your Business Lists and stay retrievable via `GET /get-scrape-results`. Only jobs still `pending`, `queued`, `running`, `processing`, or `interrupted` are cancellable — jobs already `done`, `failed`, or `cancelled` are returned in `already_terminal` untouched.

### Request

```bash
# Cancel one job
curl -X POST "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/cancel-scrape-job" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"task_id": "uuid-here"}'

# Cancel many at once (up to 500 per call)
curl -X POST "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/cancel-scrape-job" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"task_ids": ["uuid-a", "uuid-b", "uuid-c"]}'
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| task_id | string | Either this or `task_ids` | Single `task_id` returned by `/maps-search`. |
| task_ids | string[] | Either this or `task_id` | Up to 500 `task_id`s to cancel in one call. |

### Response (200 OK)

```json
{
  "success": true,
  "cancelled": ["uuid-a", "uuid-b"],
  "already_terminal": [{ "task_id": "uuid-c", "status": "done" }],
  "not_found": [],
  "message": "Cancelled 2 scrape jobs. Partial results already scraped are preserved in your Business Lists and remain retrievable via GET /get-scrape-results."
}
```

**Notes:**
- Cancellation is asynchronous — the finalizer runs in the background to persist partial results and flip any child work-tiles to `cancelled`. You can immediately re-poll `GET /get-scrape-results?task_id=...` to fetch what was scraped up to the cancel point.
- API keys can only cancel jobs owned by their user (workspace-member keys can cancel jobs they created; unknown/foreign IDs come back in `not_found`).
- Safe to call twice on the same `task_id` — the second call returns it in `already_terminal` with `status: "cancelled"`.

---

## POST /scrape-reviews-api

Scrape Google Maps reviews for businesses you already have Place IDs for (returned by `/maps-search` and `/get-scrape-results`). This endpoint is **synchronous** — reviews are returned in the response body.

### Request

```bash
curl -X POST "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/scrape-reviews-api" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"place_ids": ["ChIJ...", "ChIJ..."], "max_reviews": 20, "rating_min": 1, "rating_max": 5}'
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| place_id | string | Either this or `place_ids` | A single Google Place ID. |
| place_ids | string[] | Either this or `place_id` | Up to 10 Place IDs per call. |
| max_reviews | number | No | Max reviews per business, 1-40 (default 20). Newest first. |
| rating_min | number | No | Only return reviews with rating >= this (1-5, default 1). |
| rating_max | number | No | Only return reviews with rating <= this (1-5, default 5). |

### Response (200 OK)

```json
{
  "status": "completed",
  "requested": 2,
  "total_reviews": 34,
  "duration_ms": 3120,
  "results": [
    {
      "place_id": "ChIJ...",
      "review_count": 20,
      "reviews": [
        {
          "user_name": "Jane D.",
          "rating": 5,
          "review_text": "Great service, fast turnaround.",
          "review_time": "2 months ago"
        }
      ]
    }
  ]
}
```

**Notes:**
- Rating filtering is applied after fetching, so a narrow range can return fewer than `max_reviews`.
- A business with no reviews returns `review_count: 0` and an empty array — the call still succeeds.
- Errors: `401` invalid/missing key, `400` missing `place_id` or more than 10 IDs, `429` per-minute limit reached.
- Reviews are not billed as enrichment; the call counts against your normal API request limits.

---

## POST /location-enrich-api

Find out how many physical locations a business operates. We read the company's website (sitemap, store-locator pages, JSON-LD `LocalBusiness` entries, distinct street addresses) and fall back to a Google search when the site is inconclusive. Known large chains and franchises are detected and skipped for free.

This endpoint is **asynchronous** — submit a batch, then poll with the returned `job_id`.

### Request

```bash
curl -X POST "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/location-enrich-api" \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
        "businesses": [
          { "name": "Joe'''s Plumbing", "website": "joesplumbing.com", "country": "United States" }
        ],
        "callback_url": "https://your-app.com/webhooks/locations"
      }'
```

### Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| businesses | array | Yes | Up to 10,000 per call. Each item needs `name` or `website`. |
| businesses[].name | string | — | Business name. |
| businesses[].website | string | — | Website or domain. Strongly improves accuracy. |
| businesses[].country | string | No | Helps disambiguate the search fallback. |
| businesses[].place_id | string | No | Google Place ID, echoed back in results. |
| callback_url | string | No | POSTed when the job completes. |

### Response (202 Accepted)

```json
{
  "status": "processing",
  "job_id": "0f0e2c1a-...",
  "total_businesses": 1200,
  "unique_businesses": 940,
  "cached": 310,
  "billable": 630,
  "estimated_cost_usd": 1.26
}
```

### Poll results

```bash
curl "https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/location-enrich-api?job_id=0f0e2c1a-..." \
  -H "x-api-key: YOUR_API_KEY"
```

```json
{
  "status": "completed",
  "progress": 100,
  "billed_businesses": 630,
  "results": [
    {
      "business_name": "Joe's Plumbing",
      "website": "joesplumbing.com",
      "place_id": "ChIJ...",
      "locations": "14",
      "location_count": 14,
      "count_type": "exact",
      "confidence": "high",
      "source": "site:https://joesplumbing.com/locations"
    }
  ]
}
```

**Notes:**
- `locations` is a display string: an exact count (`"14"`), a range (`"8–12"`), `"100+"`, `"Large chain / franchise"`, or `"Unknown"`.
- Nothing is invented — thin evidence returns `"Unknown"` instead of a guess.
- **Billing: $0.002 per business analysed.** Duplicates within a submission and businesses analysed before (by any account) are free.

---






# Location Resolution

The `location` field in `/maps-search` supports:
- **City + State**: "Austin, TX", "Denver, Colorado"
- **City + Country**: "London, UK", "Toronto, Canada"
- **State/Region**: "California", "Texas" (searches top cities by population)
- **Country**: "Germany", "Australia" (searches top cities by population)
- **US ZIP code**: "90210", "78701" (geocoded via Nominatim)
- **UK postcode**: "EH1 1AB", "SW1A 1AA" (geocoded via Nominatim)

Multiple locations can be combined in a single request:
```json
{ "location": ["Austin, TX", "Denver, CO", "90210"] }
```

---

# Clay Integration Example

Use an **HTTP Request** action in Clay with a `callback_url` pointing to a Webhook source:

> ⚠️ **Important:** In Clay, all values in the request body **must be wrapped in parentheses** — even when referencing direct column values. For example, use `(/Company Name)` instead of `/Company Name`. Without parentheses, Clay will send the raw text instead of the resolved column value.

**Maps Search:**
```json
{
  "query": "(/keyword)",
  "location": "(/city), (/state)",
  "limit": 500,
  "filters": { "has_website": true, "min_rating": 3.5 },
  "callback_url": "https://your-clay-webhook-url"
}
```

**Enrichment:**
```json
{
  "business_name": "(/company_name)",
  "business_website": "(/website)",
  "want_email": true,
  "want_general_email": false,
  "general_email_fallback_only": false,
  "clay_row_id": "(/_row_id)",
  "callback_url": "https://your-clay-webhook-url"
}
```

---

# Full Example (Search + Enrich)

```javascript
// Step 1: Start a maps search
const searchRes = await fetch("https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/maps-search", {
  method: "POST",
  headers: { "Content-Type": "application/json", "x-api-key": "YOUR_API_KEY" },
  body: JSON.stringify({ query: "dentists", location: "Chicago, IL", limit: 200 })
});
const { task_id } = await searchRes.json();

// Step 2: Poll for results
let results;
while (true) {
  const poll = await fetch(
    `https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/get-scrape-results?task_id=${task_id}`,
    { headers: { "apikey": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InV1bnJsZmZvbm52bWJzc3RrcHplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzA0OTIwMzMsImV4cCI6MjA4NjA2ODAzM30.8Ic_U17YrZtsDVnYnIP0XJzrhZJOLDSlJ0AZfTS31FQ" } }
  );
  results = await poll.json();
  if (results.status === "success") break;
  await new Promise(r => setTimeout(r, 15000)); // poll every 15s
}

// Step 3: Enrich each business
for (const biz of results.data) {
  if (!biz.website) continue;
  await fetch("https://uunrlffonnvmbsstkpze.supabase.co/functions/v1/enrich", {
    method: "POST",
    headers: { "Content-Type": "application/json", "x-api-key": "YOUR_API_KEY" },
    body: JSON.stringify({
      business_name: biz.name,
      business_website: biz.website,
      business_location: biz.full_address,
      want_email: true,
      want_general_email: false,
      callback_url: "https://your-webhook-url.com/enrichment"
    })
  });
}
```

---

# Error Responses

Every response returns a JSON body — **read it regardless of the status code**. Errors contain a machine-readable `error` string plus a human-readable `message`.

| Status | `error` | Description | Retryable? |
|--------|-------|-------------|---|
| 202 | — | **Accepted.** Job queued; returns `job_id` / `task_id`. Not an error. | n/a |
| 400 | Missing required fields | Required fields not provided | ❌ Fix the payload |
| 400 | Too many keywords | More than 100 keywords in maps-search | ❌ Fix the payload |
| 400 | Too many locations | More than 100 locations in maps-search | ❌ Fix the payload |
| 400 | No valid locations found | Could not resolve any provided locations | ❌ Fix the payload |
| 401 | Missing API key | No `x-api-key` header provided | ❌ Add the header |
| 401 | Invalid API key | The key is unknown or **deactivated**. Nothing is queued and nothing is billed. | ❌ See Troubleshooting below |
| 402 | `api_key_spend_cap_reached` | This key hit its monthly spend cap. Response includes `spent_cents` and `cap_cents`. | ❌ Raise/disable the cap |
| 202 | `usage_balance_depleted` | Balance is $0.00 — the record is **held in the queue, not rejected**. Nothing is lost. | ✅ Starts automatically after top-up |
| 202 | `subscription_past_due` | Subscription payment failed — records are **held, not dropped** | ✅ Resumes after payment |
| 429 | `rate_limit_minute` | Sustained throughput exceeded the per-minute limit. `Retry-After: 60`. | ✅ **Always retry** |
| 429 | `rate_limit_daily` | Daily request limit reached (resets midnight UTC) | ✅ Retry tomorrow |
| 429 | `rate_limit_monthly` | Monthly request limit reached (resets on the 1st) | ✅ Retry next cycle |
| 500 | Internal server error | Transient platform fault | ✅ Up to 3× with backoff |

> **A 429 or 500 means the record was never ingested.** It does not enter the queue, does not trigger a webhook, and does not appear in usage analytics. If you don't retry, that lead is silently lost.

## Troubleshooting: submissions suddenly stop

If your pipeline stops landing records while your own code reports no errors, work this list in order:

1. **Check for 401s first.** The most common cause of a partial or total outage is a worker still using a **rotated or deactivated key** — some processes succeed while others fail, or throughput drops to zero with no visible error. Rejected key attempts are logged in your usage analytics with the key prefix and a reason of `unknown_api_key` or `deactivated_api_key`. Redeploy every worker with the current key, and only deactivate an old key once all traffic has moved off it.
2. **Log the status code and body of every non-2xx response.** Don't swallow failures in a `try/catch` — without client-side logs, a 401 or a dropped connection is indistinguishable from "no traffic was sent".
3. **Check for a 402.** `api_key_spend_cap_reached` rejects the call; `usage_balance_depleted` only holds it.
4. **Verify your outbound network path.** Short HTTP timeouts, connection-pool exhaustion, or DNS failures mean the request never reaches us — and anything that reaches us always produces a log entry.
5. **Check `callback_url`.** If you omit it, you will never receive a webhook and must poll instead. If you rely on webhooks, always send an explicit, publicly reachable URL that returns a 2xx.
6. **Don't assume a job is stuck.** Jobs never expire server-side. Poll `GET /get-job-status` for the true state before resubmitting — duplicates are billed twice.


## Rate Limits

> ### ⚠️ CRITICAL: Cap your client at **1000 req/min**, then retry every 429
>
> The `/enrich` and `/maps-search` endpoints enforce a per-minute throughput limit. There is **no separate concurrent-request cap** — you can burst as many parallel submissions as you like as long as your rolling 60-second total stays under the limit.

### The limit

| Limit | Default | When you hit it | 429 `error` field | `Retry-After` |
|---|---|---|---|---|
| **Per-minute throughput** | 1000 requests / rolling minute | Sustained traffic above 1000/min | `rate_limit_minute` | **60 s** |

### What a 429 looks like

```json
{
  "error": "rate_limit_minute",
  "status": 429,
  "message": "Exceeded 1000 requests/minute",
  "retry_after": 60,
  "limits": { "per_minute": { "used": 1000, "limit": 1000 } }
}
```

### What you MUST do

1. **Cap throughput at ≤ 1000 req/min** (≈ 1 request every 60 ms). Do not rely on the server to slow you down.
2. **Retry every 429.** A 429 means **your call was not accepted** — if you don't retry, that record is silently lost. It never enters the queue, never triggers a webhook, and never appears in your usage logs. Use the `retry_after` value in the response body (or the `Retry-After` header) and resend the same payload.

Without retry logic, a 10K-record run can lose **30–50% of records invisibly**.

### Polling endpoints are excluded

`get-job-results`, `get-job-status`, and `get-scrape-results` do **not** count against either limit. Poll as often as you like.

### Other notes

- Credits are only deducted when enrichment finds data. Google Maps search results count against your plan's Google Maps export allowance, not your credits.
- **Multi-title deduplication:** If you send multiple enrichment requests for the same business with different `decision_maker_title` values and the same contact person is found, you are only charged once.

### Reference: client-side throttling + retry pattern

```javascript
// ✅ CORRECT — throttles at ~460/min, retries every 429
const MIN_INTERVAL_MS = 65;      // ~920/min, safely under the 1000 ceiling

let lastSent = 0;

async function submitWithRetry(payload, attempt = 0) {
  // Throttle: never send faster than the per-minute ceiling
  const wait = Math.max(0, MIN_INTERVAL_MS - (Date.now() - lastSent));
  if (wait) await new Promise(r => setTimeout(r, wait));
  lastSent = Date.now();

  const res = await fetch(`${API_BASE}/enrich`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'x-api-key': API_KEY },
    body: JSON.stringify(payload),
  });

  if (res.status === 429) {
    if (attempt >= 8) throw new Error('Rate limited after 8 retries');
    const body = await res.json().catch(() => ({}));
    const retryAfter = body.retry_after ?? Number(res.headers.get('Retry-After')) ?? 60;
    // Add jitter so retries don't synchronize and re-collide
    const jitter = Math.random() * 500;
    const backoff = (retryAfter * 1000) + (attempt * 1000) + jitter;
    await new Promise(r => setTimeout(r, backoff));
    return submitWithRetry(payload, attempt + 1);
  }

  if (!res.ok) throw new Error(`HTTP ${res.status}: ${await res.text()}`);
  return res.json();
}
```

**For n8n / Zapier / Make:** Set the HTTP node's "On Error → Retry on Fail" with at least 5 retries and a 60s wait between retries. **Do not** mark the workflow successful when a node returns 429 — that record was rejected and must be resubmitted.

## Polling Endpoints

> **Note:** The polling endpoints use the **Supabase anon key** (via the `apikey` header), **not** your LocalPipe API key. The Supabase anon key is:
> `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InV1bnJsZmZvbm52bWJzc3RrcHplIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NzA0OTIwMzMsImV4cCI6MjA4NjA2ODAzM30.8Ic_U17YrZtsDVnYnIP0XJzrhZJOLDSlJ0AZfTS31FQ`

| Endpoint | Purpose |
|----------|---------|
| GET /get-job-results?job_id=xxx | Poll enrichment results |
| GET /get-job-status?job_id=xxx | Poll enrichment status |
| GET /get-scrape-results?task_id=xxx | Poll maps search results |

> **These are GET requests with query-string parameters — not JSON bodies.** Sending `{"job_id": "..."}` in a POST body returns `400 Missing job_id parameter`. Likewise `/maps-search` expects `query` and `location` (both accept arrays); using `keywords`/`locations` returns `400 Missing required field: query`. A `400` is never retryable — read the `message` field, fix the parameter, resend.



---

### 🚨 REQUIRED: Use Unlimited Polling

**You MUST configure your polling loop with no maximum duration.** Stop polling **only** when the response contains your record/results — never when a timer expires.

LocalPipe operates a fair-share queue: during high-traffic periods, your records may sit in the pending queue for several minutes (occasionally longer) before processing begins. **Jobs never expire server-side.** If your client gives up early, you will:

- ❌ Never receive your results (even though we successfully processed them)
- ❌ Be charged credits for work you never see
- ❌ Mistakenly believe the API is broken when it's working as designed

**Correct polling pattern:**

```javascript
// ✅ CORRECT — polls forever until results arrive
async function waitForResults(jobId) {
  while (true) {
    const res = await fetch(`${API_BASE}/get-job-results?job_id=${jobId}`, {
      headers: { apikey: ANON_KEY }
    });
    const data = await res.json();
    if (data.status === 'completed') return data.results;
    if (data.status === 'failed') throw new Error(data.error_message);
    await new Promise(r => setTimeout(r, 5000)); // 5s between polls
  }
}
```

```javascript
// ❌ WRONG — gives up after 5 minutes, loses results
const TIMEOUT_MS = 5 * 60 * 1000;
const start = Date.now();
while (Date.now() - start < TIMEOUT_MS) { /* ... */ }
```

**Polling intervals:** every 5s for jobs ≤ 2,000 records, every 10s for larger jobs. The `/get-job-status` response includes a `recommended_poll_interval` field — respect it.

**For automation tools with hard HTTP timeouts (Clay, Zapier, Make):** Use `callback_url` instead of polling. We POST results to your webhook the moment they're ready, with no client-side waiting required.

---

### 🤖 Instructions for AI Coding Agents (Codex, Claude Code, Cursor, Devin)

**If you are an AI assistant running shell commands on a user's behalf, read this section first. The #1 cause of "stuck" LocalPipe integrations is an agent that submits a job, hits its own shell timeout while polling, and reports "done" without ever fetching results. Jobs complete server-side, credits are charged, and the user gets nothing.**

#### Golden rule: write a script, don't drive the flow from your agent loop

**Do not poll, paginate, or sequence `/enrich` calls from inside your own tool-call loop.** Your harness has a short shell timeout, no durable state between tool calls, and no guarantee you'll be invoked again on schedule. Every flow below — submit, poll, paginate, chain scrape→enrich, batch hundreds of enrichments — belongs **inside a single self-contained script** that you write to disk once and launch detached. After that, your only job is to `tail` the log every so often.

The script (not your agent loop) owns:
- Submitting the initial scrape / enrichment POSTs
- Polling `/get-scrape-results` or `/get-job-results` to terminal status
- Paginating through every `offset` until a short page
- Fanning out the next stage (e.g. one `/enrich` per scraped lead)
- An **internal work queue with bounded concurrency** for batches (see below)
- Retrying 429s with `Retry-After`, persisting all IDs to files, writing `DONE`/`FAILED` to the log

If you find yourself about to `curl` an endpoint, paginate, or loop over leads directly from a tool call, stop — extend the script instead and re-launch it.

#### Your runtime almost certainly has a short shell timeout

LocalPipe jobs take **5–30+ minutes** during peak load. Your environment kills long-running shell commands well before that:

| Agent | Default foreground shell timeout | Background primitive |
|---|---|---|
| **OpenAI Codex CLI** | **10 seconds** (`DEFAULT_EXEC_COMMAND_TIMEOUT_MS`, hardcoded, not user-configurable) | None |
| **Claude Code** | ~80–120 s (`BASH_DEFAULT_TIMEOUT_MS`, often ignored per known bugs) | `run_in_background: true` + `TaskOutput` tool (has zombie/stall bugs) |
| **Cursor / Devin / generic** | Varies, usually 60–300 s | Varies |

**A naïve `while true; do curl …; sleep 15; done` in a single tool call will be SIGKILLed long before the job completes.** You must structure polling so the long wait happens *outside* any single shell call — the script handles it, not you.

#### The only reliable pattern: detached script + log file + short poll calls

Do all three steps in this order. Do not skip step 1.

**Step 1 — Write a self-contained poll script to disk:**

```bash
cat > /tmp/localpipe_poll.sh << 'EOF'
#!/usr/bin/env bash
set -u
API_BASE="https://uunrlffonnvmbsstkpze.supabase.co/functions/v1"
API_KEY="$LOCALPIPE_API_KEY"
ANON_KEY="<supabase-anon-key-from-docs-above>"

# Submit the scrape
TASK_ID=$(curl -sS -X POST "$API_BASE/maps-search" \
  -H "x-api-key: $API_KEY" -H "content-type: application/json" \
  -d '{"query":["dentist"],"location":["Austin, TX","Dallas, TX","Houston, TX","San Antonio, TX","Fort Worth, TX"],"limit":500}' | jq -r '.task_id')
echo "TASK_ID=$TASK_ID" | tee /tmp/localpipe_task.id

# Poll forever — exit ONLY on terminal status
OFFSET=0
while true; do
  RESP=$(curl -sS -H "apikey: $ANON_KEY" \
    "$API_BASE/get-scrape-results?task_id=$TASK_ID&limit=5000&offset=$OFFSET")
  STATUS=$(echo "$RESP" | jq -r '.status')
  COUNT=$(echo "$RESP" | jq -r '.results | length')
  echo "[$(date -u +%H:%M:%S)] status=$STATUS offset=$OFFSET batch=$COUNT"
  if [ "$STATUS" = "done" ]; then
    echo "$RESP" > "/tmp/localpipe_results_$OFFSET.json"
    [ "$COUNT" -lt 5000 ] && break
    OFFSET=$((OFFSET + 5000))
    continue
  fi
  [ "$STATUS" = "failed" ] && { echo "FAILED"; exit 1; }
  sleep 15
done
echo "DONE"
EOF
chmod +x /tmp/localpipe_poll.sh
```

**Step 2 — Launch it detached so it survives your shell timeout** (this command returns instantly):

```bash
nohup /tmp/localpipe_poll.sh > /tmp/localpipe.log 2>&1 &
echo $! > /tmp/localpipe.pid
disown
echo "started pid=$(cat /tmp/localpipe.pid)"
```

**Claude Code:** prefer the native primitive — call `Bash` with `run_in_background: true` on the `/tmp/localpipe_poll.sh` invocation, capture the returned job ID, and use `TaskOutput` in step 3 instead of `tail`. Fall back to `nohup` if `TaskOutput` returns stale output.

**Step 3 — Across separate, short tool calls, poll the log file until you see `DONE` or `FAILED`:**

```bash
# Each of these is a separate Bash tool call — well under any agent's timeout.
# Repeat every 30–60s until "DONE" appears in the log.
tail -20 /tmp/localpipe.log
ps -p "$(cat /tmp/localpipe.pid)" > /dev/null && echo "still running" || echo "exited"
```

When the log shows `DONE`, read `/tmp/localpipe_results_*.json` and present the results. **Do not end the user's turn until either `DONE` or `FAILED` appears in the log.**

#### Chained workflows (scrape → enrich)

Extend the same script: after the scrape `DONE` line, **count the total scraped leads across every `/tmp/localpipe_results_*.json` page** and write it to `/tmp/localpipe_scrape_total`. Then iterate every business in every page and `POST /enrich` for each, appending each returned `job_id` to `/tmp/localpipe_enrich_jobs.txt`. **Before moving on, assert `wc -l /tmp/localpipe_enrich_jobs.txt` equals the scrape total** — if it's less, you dropped leads (almost always a forgotten `offset` page or a hit shell timeout mid-loop). Resume from the missing offset and re-submit. Only after the counts match should you poll the enrichment jobs via `/get-job-results`. All of this stays inside the one detached script so a single `nohup` covers the whole chain.

> **Common silent failure:** a scrape returns 5,000+ leads across multiple pages but the agent submits enrichment for only the first 400, then exits because Codex's 10s timer killed the loop. The user sees "done" with 92% of their data missing and no error. The count assertion above is the only thing that catches this.

#### Batching many scrapes or enrichments? Queue them inside the script

Anytime the user's request fans out into more than a handful of `/maps-search` or `/enrich` calls (e.g. "scrape 20 cities", "enrich this 5K CSV", "run these 50 keyword × city combos"), **do not submit them serially from your tool calls and do not fire them all at once with `&`**. Both lose data the moment your shell call dies or you hit the 1000 req/min rate limit. Instead, the script should run a tiny in-memory work queue:

```bash
# Inside /tmp/localpipe_poll.sh — append to a work file, drain with bounded concurrency
WORK=/tmp/localpipe_work.txt        # one JSON payload per line (scrape or enrich input)
DONE=/tmp/localpipe_submitted.txt   # task_id / job_id per accepted submission
CONCURRENCY=8                       # safe under 1000/min for /enrich and /maps-search

drain_queue() {
  : > "$DONE"
  while IFS= read -r PAYLOAD; do
    # background each submission, cap parallelism
    while [ "$(jobs -rp | wc -l)" -ge "$CONCURRENCY" ]; do sleep 0.2; done
    (
      RESP=$(curl -sS -X POST "$API_BASE/enrich" \
        -H "x-api-key: $API_KEY" -H "content-type: application/json" \
        -w "\n%{http_code}" -d "$PAYLOAD")
      CODE=$(echo "$RESP" | tail -1)
      BODY=$(echo "$RESP" | sed '$d')
      if [ "$CODE" = "429" ]; then
        sleep 60                                  # honor Retry-After
        echo "$PAYLOAD" >> "$WORK.retry"          # requeue
      elif [ "$CODE" = "202" ] || [ "$CODE" = "200" ]; then
        echo "$BODY" | jq -r '.job_id // .task_id' >> "$DONE"
      else
        echo "$PAYLOAD" >> "$WORK.failed"         # surface in log, do not silently drop
      fi
    ) &
  done < "$WORK"
  wait
  # re-drain retries until empty
  [ -s "$WORK.retry" ] && { mv "$WORK.retry" "$WORK"; drain_queue; }
}
```

Why this matters:

- **No data loss on agent timeout** — the queue lives on disk; the detached script keeps draining it after your tool call dies.
- **Stays under 1000/min** — bounded concurrency + 429 requeue is the only safe way to submit hundreds of jobs without losing them. A naked `for` loop with `&` will burst-trigger 429s and silently drop the rejected ones.
- **Resumable** — if anything crashes, the script restarts and resumes from `$WORK` (whatever wasn't moved to `$DONE` is still pending).
- **Auditable** — `wc -l "$DONE"` must equal `wc -l "$WORK_ORIG"` before you tell the user "submitted". Same count-matching rule as the chained scrape→enrich flow.

For scrapes that fan out across many cities/keywords, write each `{keyword, city, state}` combo as one line in `$WORK` and reuse the exact same `drain_queue` against `/maps-search` — then feed every returned `task_id` into a second queue that polls them all to completion.

#### Easier alternative — let us push to you (zero polling)

If the user has any webhook-capable destination (their own server, webhook.site, n8n/Make/Zapier webhook URL, ngrok tunnel), include `callback_url` in both `/maps-search` and `/enrich` POSTs. The script can then exit immediately after enqueueing — we POST the full result payload the moment each job completes. **Always prefer this over polling when possible.** You still need the script + queue for submission and 429 retries, but you can skip the polling loop entirely.

#### Hard rules — never violate these

1. **Never submit and exit.** A `POST` without either (a) a launched detached poll script or (b) a `callback_url` = guaranteed data loss for the user.
2. **Never put the long wait inside a single foreground tool call.** Codex's 10s timer will kill it; Claude's ~80s timer will kill it; the user sees a hang and you see a SIGKILL.
3. **Never add `maxAttempts` or `timeoutMs` to the poll loop.** Exit only on `status: "done" | "completed" | "failed"`. Jobs legitimately run 30+ minutes.
4. **Always persist `task_id` / `job_id` to a file** (`/tmp/localpipe_task.id`) *before* starting the poll. If anything crashes, the user can resume by re-running step 3 against the saved ID.
5. **Always tail the log between poll calls** and show the user the latest status line — silent waiting looks like a hang and users will Ctrl-C.
6. **202 is success, not an error.** Some HTTP clients fail `res.ok` checks on 202. Always read the JSON body regardless of status code on `/enrich` and `/maps-search`.
7. **Always paginate `/get-scrape-results` until `results.length < limit`.** A single call with `limit=5000` is NOT guaranteed to return everything — keep incrementing `offset` until you get a short page. Track total leads received.
8. **Enrichment submission count MUST equal scrape result count.** Before declaring a chained workflow complete, assert `enrich_jobs_submitted == scrape_total`. If they don't match, find and resubmit the missing leads — do not tell the user the run is done.
9. **Never sequence multi-step flows from your agent loop.** Submit → poll → paginate → fan-out → poll-again all belong inside the script. If your plan involves more than one tool call making HTTP requests to LocalPipe, you wrote the script wrong — extend it instead.
10. **Always queue batches with bounded concurrency inside the script.** No `for lead in leads; do curl …; done` (serial = times out, drops the tail), no `for lead in leads; do curl … &; done` (burst = 429s, drops the rejected). Use the `drain_queue` pattern above.

#### Anti-patterns — never generate this code

- ❌ `curl -X POST .../enrich && echo "submitted, check dashboard"` and end the turn
- ❌ `while true; do curl .../get-job-results; sleep 15; done` as a single tool call (killed by shell timeout)
- ❌ `for i in $(seq 1 20); do …; sleep 15; done` (gives up after 5 min)
- ❌ Polling inside a detached `setTimeout`/Promise after the main function returns
- ❌ Submitting 100 enrichments in a loop without launching any poller for them
- ❌ Treating 202 Accepted as a failure
- ❌ `curl ".../get-scrape-results?task_id=X&limit=5000" | jq '.results[] | …enrich…'` with no `offset` pagination — silently drops everything past the first page
- ❌ Reporting "enrichment started for N leads" where N is smaller than the scrape's total
- ❌ Driving the scrape→poll→enrich→poll chain across multiple of your own tool calls instead of putting it in one detached script
- ❌ `for lead in $(cat leads.json); do curl .../enrich … &; done; wait` — unbounded fan-out; the 429s vanish into the void
- ❌ `for city in cities; do curl .../maps-search -d '{"query":"plumber","location":"'$city'"}'; done` — `/maps-search` accepts arrays for both `query` and `location`; send ONE request with `"location": [...all cities...]` (up to 100) and `"query": [...all keywords...]` (up to 100). Looping per-city is the #1 way AI agents waste rate limit and drop data.
- ❌ Sending one request per keyword because you are worried about budget skew — each pair now gets its own share of `limit`, so batch up to 100 keywords in one `query` array and raise `limit` to `N × pairs` if you need a guaranteed count each.
- ❌ Calling a run done without reading `sub_query_breakdown`. A keyword or city with `found: 0, truncated: false` is a genuinely empty market; do not re-run it. Only `truncated: true` means the `limit` cut results off; raise `limit` or use `"limit": "all"`.
- ❌ Treating a zero-row result for a **custom keyword** as an empty market. Custom keywords bypass exact category matching and use relevance search, so test with `"limit": "all"` or a larger limit before deciding no leads exist.




## Best Practices

1. **Always batch locations.** Pass every city in a single `/maps-search` call (up to 100 locations). It is MUCH faster, consumes one rate-limit slot, and avoids the data loss that comes from looping per-city.
2. **Batch keywords too.** With an integer `limit`, the budget is split evenly across every keyword × location pair (unused share is redistributed), so batching never starves later keywords. Need N per pair? Set `limit = N × pairs`.
3. **Cap your client at ≤ 1000 requests/minute** — anything above this is rejected with 429 and lost unless you retry
4. **Always retry 429 responses** — honor `Retry-After` (60s default), then back off exponentially. A 429 means the call was *not* accepted
5. **Always use callback_url** for Clay and automation platforms — avoids timeout issues
6. **Split large locations** — searching "Texas" searches top 50 cities; for full coverage, list specific cities
7. **Use filters** to reduce noise — `has_website: true` and `min_rating: 3.5` dramatically improve lead quality
8. **Poll every 15 seconds** — most small searches complete in under 60 seconds
9. **Combine search + enrich** — use maps search to find businesses, then enrich each one for owner contact info

---

# Troubleshooting / FAQ

## Why did some of my locations or keywords return zero results?

With multi-keyword × multi-location batches, `limit` is divided evenly across pairs and any unused share is redistributed to pairs that still have more matches. So a pair returning zero results means Google had no matching businesses for that exact combination. If most pairs still look truncated, your `limit` is simply too small for the number of pairs — raise it or use `"limit": "all"`.

Verify this by reading `sub_query_breakdown` in the completed response (returned for multi keyword × location tasks with an integer `limit`; single-pair tasks and `"limit": "all"` tasks omit it):

```json
{
  "sub_query_breakdown": {
    "by_location": [
      { "location": "Sydney, Australia", "found": 505, "returned": 300, "truncated": true },
      { "location": "Perth, Australia", "found": 0, "returned": 0, "truncated": false }
    ],
    "by_keyword": [
      { "keyword": "dermatology clinic", "found": 505, "returned": 300, "truncated": true }
    ]
  }
}
```

- `found: 0, truncated: false` → no matches for that location/keyword. Do not re-run it separately.
- `found: > 0, truncated: true` → matches exist but the task hit its global `limit`. Raise `limit` (e.g. `N × pairs`) or switch to `"limit": "all"` for full coverage.
- `found: > returned, truncated: true` → normal behavior for an integer cap.

If you see `truncated: true` on every pair, your `limit` is simply lower than the available density in those areas.

## Should I split a batch into one request per city or keyword?

No. Batch both locations and keywords (up to 100 of each per call). With an integer `limit` the budget is split evenly across pairs and unused share is redistributed, so no keyword or city is starved. If you need more total results, raise `limit` (max 50,000) or use `"limit": "all"`. Use `sub_query_breakdown` to tell truncation apart from a genuinely empty pair.

---

# Infrastructure

## Queue-Based Processing

All enrichment requests are **asynchronously queued** for processing. When you submit a request:

1. **Immediate acceptance** — Every valid request returns 202 Accepted instantly
2. **Queue processing** — Records are processed from the queue in batches
3. **Webhook delivery** — Results POST to your callback_url when complete

This architecture allows you to send thousands of requests simultaneously without waiting, **as long as your client throttles to ≤ 1000/min** and retries any 429s.

## Rate Limiting Structure

To ensure stability during traffic spikes, the API implements tiered rate limits:

| Limit | Default | Applies To | Behavior |
|-------|---------|------------|----------|
| Per-minute | **1000 requests** | /enrich, /maps-search | Returns **429** with `Retry-After: 60` header — **client MUST retry** |
| Daily | **Unlimited** | All endpoints | No daily cap |
| Monthly | **Unlimited** | All endpoints | No monthly cap |

> **Why this matters:** 429s short-circuit before the request reaches the queue. They are **not logged in your usage analytics**. If your client doesn't retry, those records are lost — no job created, no webhook fired, no error visible in the dashboard.

**Polling endpoints are excluded from all rate limits:** get-job-results, get-job-status, and get-scrape-results do not count against your quota.

**Custom limits:** High-volume users can request increased per-minute limits by contacting support.

## Database Optimization

Recent infrastructure improvements have reduced database operations per request by 50%:
- Consolidated credit checking and subscription validation
- Asynchronous API logging (fire-and-forget pattern)
- Optimized batch processing via cron-based scheduling

These changes ensure consistent performance even during peak usage periods.
