---
title: "LocalPipe data quality and the October 2026 benchmark"
summary: "Average find rates, what verified means, what to expect per 1,000 businesses, and the full five-provider benchmark on 500 US plumbing businesses with its method and limits."
verified_against: "https://www.localpipe.io/data-comparison, /faq, /about, /llms-full.txt"
verified_on: "2026-11-13"
---

# LocalPipe data quality and the October 2026 benchmark

There are two sets of numbers. Keep them apart when quoting.

1. **Average find rates** across local, owner-operated businesses (LocalPipe internal benchmark averages).
2. **The October 2026 benchmark**, a single published head-to-head on one list of 500 US plumbing businesses.

## 1. Average find rates

Measured on local, owner-operated businesses, which is the segment LocalPipe is built for. They are not representative of corporate B2B lists.

| Measure | Average |
| --- | --- |
| Owner name find rate | **75%** |
| Combined email find rate (business + owner emails) | **60%** |
| Owner phone find rate | **20%** |
| Bounce rate reported by users in live campaigns | **Sub 1%** (0.11% in one documented case) |
| Sources cross-referenced | 11+ |
| Emails sent by users on LocalPipe data | 34,700,000+ |

### What to expect from 1,000 local businesses

Roughly **750 owner names**, **600 emails** across business and owner addresses combined, and **200 direct owner phones**. Expect a partial return, not a full one. Billing is pay on find, so you are charged for records that return data, not for all 1,000 rows.

What moves these numbers:

- Tight categories of owner-operated businesses (roofers, plumbers, dentists, restaurants) tend to land at or above the averages.
- Franchises, chains and businesses with no website tend to fall below.
- Rates vary by country, because public record sources differ outside the United States.

The playbook splits the email figure further: about **42% owner direct email** and about **60% combined** once generic addresses are counted. See [15.2 Benchmarks With Confidence](../playbook/15-reference/15.2-benchmarks-with-confidence.md).

### What "verified" means

Verified means the address passed validation before it was returned: syntax, domain and mailbox checks were run, by three independent verification services, rather than an address being guessed from a pattern and handed over untested. Verification is included in the price.

Two caveats the site states itself:

- Validation confirms an address is deliverable, not that the person still works there. A list left sitting for months will decay.
- A business email such as info@ may be deliverable while still reaching a receptionist rather than the owner. That is why business and owner emails are priced and labelled separately.

### Owner phones

The owner phone rate is low (20%) because personal numbers are far less likely to be published anywhere public, and they cost more (4 credits). If phone is the primary channel, the main business number from the Google Maps listing is available on effectively every record. A common approach is to call the main line and use the returned owner name to ask for the right person.

## 2. The October 2026 benchmark

**Source:** https://www.localpipe.io/data-comparison. Published 11 October 2026, last reviewed 12 November 2026. Published by LocalPipe.

**Always describe it as a vendor-run benchmark.** LocalPipe ran it and is one of the five providers tested. The sample, definitions, rounding rules and scoring formula are published so it can be repeated.

### Headline

On 500 US plumbing businesses, LocalPipe returned a verified, sendable email for **51.6%** (258). The next best provider, Outscraper, reached 12.0% (60). Seamless, Apollo and Hunter reached between 4.5% and 7.0%. LocalPipe named the owner for **76%** and returned a direct mobile for **16.6%**. No other provider named more than 16% of owners.

Against Apollo specifically: 258 sendable emails against 26 on the same list, about **10x** (the homepage states 9.9x, from 516 against 52 per 1,000). A team using Apollo would need to source roughly 5,000 plumbing businesses to end up with as many sendable contacts as LocalPipe returned from 500.

### Scorecard

| Provider | Benchmark Score | Verified sendable coverage | Email find rate | Owner name found | Valid emails | Invalid emails shipped | Catch-all or risky | Direct owner mobile |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **LocalPipe** | 100.0 | 51.6% (258/500) | 52% (260) | 76% (380/500) | 99.2% (258 of 260) | 0% (0) | 0.8% (2) | 16.6% (83/500) |
| Seamless | 30.1 | 7.0% (35/500) | 8.1% (40) | 10.1% (50/500) | 87% (35 of 40) | 7% (3) | 6% (2) | 3.7% (18/500) |
| Apollo | 26.4 | 5.2% (26/500) | 7.2% (36) | 11.2% (56/500) | 72% (26 of 36) | 18% (6) | 12% (4) | 4.1% (20/500) |
| Hunter | 23.7 | 4.5% (22/500) | 6.3% (32) | 8.2% (41/500) | 71% (22 of 32) | 16% (5) | 13% (4) | 3.1% (16/500) |
| Outscraper | 22.2 | 12.0% (60/500) | 28% (140) | 16% (80/500) | 43% (60 of 140) | 36% (50) | 21% (29) | Not provided |

Apollo's valid, invalid and catch-all shares total 102% as reported because of rounding in the source figures.

### Why the gaps exist

- **Coverage.** Apollo, Seamless and Hunter draw on professional networks, corporate websites and the email patterns of companies with structured domains. A three-truck plumbing company usually leaves none of that trail. The owner may run the business from a personal Gmail or Yahoo address, or a domain a web designer set up once. Those three clustered between 6.3% and 8.1% email coverage.
- **Verification.** Outscraper found the second most emails (28%), but 36% were invalid and 21% were catch-all. Sent as is, its output would have produced 50 hard bounces from a 500-record list.
- **Owner names.** Outscraper found emails for 28% but names for only 16%, so a large share of its addresses are generic inboxes with no person attached.

### Projected hard bounce if every returned email were sent

Invalid addresses only. Catch-alls can fail on top.

| Provider | Projected bounce |
| --- | --- |
| LocalPipe | 0% |
| Seamless | 7% |
| Hunter | 16% |
| Apollo | 18% |
| Outscraper | 36% |

Most deliverability practitioners treat 2% as the ceiling for healthy cold outreach, which is the same bar the playbook uses ([4.5](../playbook/04-deliverability/4.5-the-2-percent-bar-and-infrastructure-by-contact-bucket.md)).

### Scaled to 1,000 plumbing businesses

| Per 1,000 businesses | LocalPipe | Seamless | Apollo | Hunter | Outscraper |
| --- | --- | --- | --- | --- | --- |
| Emails returned | 520 | 81 | 72 | 63 | 280 |
| Sendable (verified valid) | 516 | 70 | 52 | 45 | 120 |
| Verified sendable coverage | 51.6% | 7.0% | 5.2% | 4.5% | 12.0% |
| Hard bounces waiting in the list | 0 | 6 | 13 | 10 | 101 |
| Catch-all addresses to triage | 4 | 5 | 9 | 8 | 59 |
| Owners identified by name | 760 | 101 | 112 | 82 | 160 |
| Owner mobiles to call | 166 | 37 | 41 | 31 | n/a |
| LocalPipe advantage, sendable emails | Baseline | 7.3x | 10.0x | 11.5x | 4.3x |

### Verdict by provider

| Provider | Sendable coverage | Best fit | Main watch-out |
| --- | --- | --- | --- |
| LocalPipe | 51.6% | Local owner-operators | Mobiles reach about 1 in 6 owners |
| Outscraper | 12.0% | Raw Google Maps data at volume | 57% of returned emails invalid or risky |
| Seamless | 7.0% | Mid-market companies | 8.1% email find rate |
| Apollo | 5.2% | Corporate and tech prospecting | 30% of returned emails invalid or risky |
| Hunter | 4.5% | Firms with domain emails | Lowest email find rate (6.3%) |

### Method

- **Sample.** 500 US plumbing businesses drawn from Google Maps listings. The same 500 records went to each provider.
- **Providers.** LocalPipe, Apollo, Seamless, Hunter, Outscraper, each through its standard enrichment or search workflow for finding contacts at a known business.
- **Verification.** Every email from every provider was independently verified. Deliverable counted as valid. Failed-despite-being-returned counted as invalid. Addresses on catch-all domains (which MillionVerifier classifies as risky) were counted separately.
- **Rounding.** Counts are derived from measured rates and rounded to the nearest whole record.

| Metric | Definition |
| --- | --- |
| Email find rate | Share of the 500 businesses for which the provider returned at least one email |
| Owner name find rate | Share for which the provider identified the owner or primary decision maker by name |
| Valid emails | Share of returned emails that verified as deliverable |
| Invalid shipped | Share of returned emails that failed verification despite being returned as valid |
| Catch-all or risky | Share of returned emails on domains that accept all mail |
| Mobile coverage | Share of businesses for which a direct owner mobile was returned |
| **Verified sendable coverage** | Email find rate × valid rate. The share of the full list that ends with a verified, deliverable email |
| Benchmark Score | 40% verified sendable coverage, 20% owner name rate, 20% valid rate, 20% mobile coverage, each scaled to the best result in the test |

### Limits to state with it

- Single vertical, 500 records. Other industries and regions may differ, and provider databases change.
- Vendor-run. Treat it as a strong directional signal and validate on your own market.
- The 52% email and 76% owner name results are close to LocalPipe's published averages, which suggests plumbing was not an unusually easy sample.

### The five recommendations the report closes on

1. Measure verified sendable coverage, not find rate.
2. Use a tool built for the segment.
3. Lead with the owner's name.
4. Pair email with phone, following calling and texting rules.
5. Run your own test on a few hundred records from your own market.

### How to cite

> LocalPipe. LocalPipe Data Quality and Coverage Benchmark: 5 Providers Compared. October 2026. https://www.localpipe.io/data-comparison
