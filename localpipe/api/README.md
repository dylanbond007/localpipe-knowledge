---
title: "LocalPipe API quick reference"
summary: "What the API does, the endpoints, the rules that prevent lost data (batch locations, retry 429s, poll until terminal), and where the full docs and category list live."
verified_against: "https://www.localpipe.io/localpipe-api-docs.md and https://www.localpipe.io/localpipe-business-categories.csv"
verified_on: "2026-11-13"
---

# LocalPipe API quick reference

This page is a map. For request and response shapes, read the full docs.

| File | What it is |
| --- | --- |
| [api-docs.md](api-docs.md) | Unmodified copy of the published API docs, fetched 2026-11-13. Written for an agent to read. About 1,550 lines |
| [business-categories.csv](business-categories.csv) | Every business category the system recognises, 4,284 rows, sourced from Google Maps' taxonomy. Use it to see which keywords trigger exact category matching in `/maps-search` |

**The live copies are the source of truth.** Re-fetch before building anything that depends on endpoint details:

- https://www.localpipe.io/localpipe-api-docs.md
- https://www.localpipe.io/localpipe-business-categories.csv

## What it is

A RESTful HTTP API authenticated with one API key in the `x-api-key` header, with webhook callbacks. API access is included on every plan. About 90% of high-volume users run through the API rather than the interface, and one API call replaces a seven-step, five-table Clay enrichment flow.

Everything is **asynchronous**. A request returns `202 Accepted` with a job or task id straight away. Results arrive at your `callback_url` webhook, or you poll for them.

## Endpoints

| Endpoint | Purpose |
| --- | --- |
| `POST /enrich` | Enrich one business with owner or decision-maker contact info: owner name, owner email, general business email, owner phone |
| `POST /cancel-enrich-job` | Cancel one or many enrichment jobs (up to 500 per call) |
| `POST /maps-search` | Search Google Maps by keyword and location, with filters. Batches up to 100 keywords and up to 100 locations in one request |
| `POST /cancel-scrape-job` | Cancel one or many search jobs |
| `GET /get-usage` | Account type, remaining credits and remaining Google Maps exports for the billing cycle |
| `GET /list-business-lists` | List your saved Business Lists and their IDs |
| `/business-list-api` | Actions on an existing Business List: read, merge, move, edit, remove, enrich, delete, scrape reviews |
| `POST /scrape-reviews-api` | Scrape Google reviews |
| `POST /location-enrich-api` | Location enrichment |
| Polling: `get-job-results`, `get-job-status`, `get-scrape-results` | Fetch results. These do not count against rate limits |

## Enrichment: the fields that matter

- `business_name` is the only required field. `business_website` and `business_location` improve accuracy.
- `want_name` (default true), `want_email` (default true, 2 credits), `want_phone` (default false, 4 credits), `want_general_email` (default false).
- `general_email_fallback_only`: only return the generic business email when no owner email is found. This is how the playbook's contact buckets get built without paying twice. See [6.4 The Four Contact Buckets](../../playbook/06-building-the-list/6.4-the-four-contact-buckets.md).
- `decision_maker_title`: target a specific role instead of the owner. This is custom title waterfalling. Sending several titles for the same business is charged once if the same person is found.
- `owner_name`: supply a name you already have and skip name discovery.
- `business_phone`: recommended when requesting owner phones.
- `clay_row_id` and `callback_url`: for matching results back to rows in Clay or another automation platform.

## Search: the fields that matter

- `query`: a keyword or an array of up to 100. Matched against Google Maps' category taxonomy. When a category matches exactly, results are filtered to businesses carrying that category, and each result reports its `matched_category`. Custom keywords still work when nothing matches.
- `location`: city, state, country, US ZIP or UK postcode, or an array of up to 100.
- `limit`: total budget for the task. Default 100, minimum 50, maximum 50,000. Split evenly across every keyword × location pair, with unused share redistributed. Pass `"all"` or `null` for Find ALL mode.
- `filters`: `min_rating`, `max_rating`, `min_reviews`, `max_reviews`, `has_website`, `has_phone`, `price_level`, `opens_before`, `closes_after`, `radius_km`.
- `exclude_existing_lists`: drop businesses already in your saved lists.

This maps directly onto the playbook's sourcing technique. Category matching, query stacking and geographic splitting are in [6.3 Category Matching and Query Depth](../../playbook/06-building-the-list/6.3-category-matching-and-query-depth.md).

## Rules that prevent silent data loss

These come from the docs and are the ones agents get wrong.

1. **Batch locations. Never loop cities.** One `/maps-search` call takes up to 100 locations and fans them out under one task. One request per city is the main cause of incomplete data and rate-limit exhaustion.
2. **Batch keywords too.** The budget splits across pairs, so later keywords are not starved. For N results per pair, set `limit = N × pairs`.
3. **Cap the client at 1,000 requests per minute.** There is no separate concurrency cap.
4. **Retry every 429.** A 429 means the call was not accepted. Without a retry that record is lost with no trace: no queue entry, no webhook, no usage log. Honour `retry_after` (60 seconds by default).
5. **Use `callback_url`** for Clay and automation platforms to avoid timeouts.
6. **Poll until a terminal status.** Do not stop on a timer. Polling endpoints are excluded from rate limits. The docs carry a dedicated section of instructions for AI coding agents on long-running jobs.
7. **Split large locations.** Searching a whole state searches its top cities. For full coverage, list specific cities.
8. **Check `GET /get-usage`** before a large job.

## Billing through the API

- Credits are deducted only when enrichment finds data.
- Maps search results count against the plan's Google Maps export allowance, not credits.
- A contact found in the last 30 days is not charged again.
- If a call or a Clay table fails, results are recoverable from the Data Recovery panel in the dashboard.

Pricing detail is in [../pricing.md](../pricing.md).

## Building with it from Claude Code

The playbook's recipe ([6.8 Building It](../../playbook/06-building-the-list/6.8-building-it.md)): give Claude Code the API docs, the category CSV and an API key, then ask it to build a search-and-enrich engine you can drive in words. Five to ten minutes.

**Never commit an API key to this repo.** Keys belong in the environment of whatever is running the job.
