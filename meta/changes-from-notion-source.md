---
title: "Changes from the Notion source"
summary: "Everything that differs between the Notion export of The Local Legend System and this repo: corrections made against localpipe.io, claims that could not be confirmed, what the site has that the guide did not, and the structural conversion."
date: "2026-11-13"
---

# Changes from the Notion source

The playbook was converted from a Notion export received on 13 November 2026 and cross-checked against https://www.localpipe.io the same day. This page records what changed and what is still open.

## 1. Corrections made to chapter text

One fact in the guide was out of date: **the free tier.** The guide said LocalPipe had a free tier of 400 credits with no card required. The site now states there is no free plan and plans start at $57 a month. Three passages carried it. Each edited passage is followed by an HTML comment in the file so it can be found again.

| File | Original wording | Now reads |
| --- | --- | --- |
| [13.2 The Data Layer Compared](../playbook/13-tech-stack-and-costs/13.2-the-data-layer-compared.md) | "Credits are consumed only on successful results, and there is a free tier of 400 credits with no card required, which makes testing it against your own market cheap enough that you should just do it rather than take anybody's word for the numbers." | "Credits are consumed only on successful results. There is no free plan (plans start at $57 a month), but because a miss costs nothing, testing it against a sample of your own market is cheap enough that you should just do it rather than take anybody's word for the numbers." |
| [13.2 The Data Layer Compared](../playbook/13-tech-stack-and-costs/13.2-the-data-layer-compared.md) | "The first two are published by LocalPipe or derived from their material, which is exactly why the disclosure above is there. Run the free tier against your own market before you believe any of it." | "The first three are published by LocalPipe or derived from their material, which is exactly why the disclosure above is there. Run a sample of your own market through it before you believe any of it." |
| [15.2 Benchmarks With Confidence](../playbook/15-reference/15.2-benchmarks-with-confidence.md) | "Both are published by or derived from LocalPipe, who co-authored the list building section of this guide. Their free tier is 400 credits with no card, so run it against your own market rather than taking any of it on trust." | "All three are published by or derived from LocalPipe, who co-authored the list building section of this guide. There is no free plan (plans start at $57 a month, charged only on finds), so run a small sample of your own market rather than taking any of it on trust." |

## 2. Additions to chapter text

Four small additions, each marked in the file.

| File | Added |
| --- | --- |
| 13.2, "Where These Figures Come From" | A third source bullet linking LocalPipe's October 2026 data benchmark (https://www.localpipe.io/data-comparison). This is why "first two" became "first three" above |
| 15.2, "Where to Check" | The same benchmark link as a third bullet. This is why "Both" became "All three" above |
| 13.2, before "How to Decide" | A repo note pointing to `localpipe/` for current plans, credit costs and the benchmark |
| [6.8 Building It](../playbook/06-building-the-list/6.8-building-it.md) | A repo note pointing to the copy of the API docs and category CSV in `localpipe/api/` |

Repo notes are labelled "Repo note, not part of the original guide".

## 3. Claims the public site does not confirm

These are in the guide, are not contradicted by the site, and could not be verified there. They were left as written. **Each one needs a yes or no from Dylan.**

| # | Claim | Where | Why it is open |
| --- | --- | --- | --- |
| 1 | Triple verification runs through **MillionVerifier, ZeroBounce and NeverBounce** | 13.2 | The site says only "three independent verification services". It names MillionVerifier once, as the tool that classifies catch-all addresses in the benchmark |
| 2 | **30% off lifetime monthly with code `LOCALLEGEND`** | [CTA library](../reference/cta-library.md), [16 Webinar Hub](../playbook/16-webinar-hub-business-owner.md) | Not listed anywhere on the public site. Is it still active, and does it apply to the current $57, $97 and $157 plans? |
| 3 | Source link to a **third-party listing** (saasworthy.com/product/localpipeio) for the verification stack and sourcing model | 13.2, 15.2 | Kept because the guide cites it. A third-party listing may still show the old free tier. Consider replacing it with a first-party page |
| 4 | **Owner direct email find rate about 42%** | 15.2, both edition front pages | The site publishes only the 60% combined figure. The 42% is the guide's controlled-comparison number |
| 5 | **Under 5% of local SMBs are in the major B2B databases**, and market figures drawn from LocalPipe's analysis of **roughly 7.2 million emails** | 0.4, 6.1, 15.2 | Guide-sourced. The site does not publish the 7.2 million analysis. The October 2026 benchmark is in the same direction (Apollo 5.2% verified sendable coverage on one list) but measures something different |
| 6 | Partner codes: **InboxKit `GTMLAUNCHPAD`** (about $3 per inbox per month) and **Sendkit `10GTMLAUNCHPAD`** (10% off) | 13.1, 16 | Third-party offers. Not checked |
| 7 | **The Local Legend Money Model** calculator (Google Sheets link) | 3.4, 16 | Link kept as written. Not opened or checked |
| 8 | "Both routes are covered in **the training that comes with this guide**" | 6.8 | No training material was in the export |

Two cosmetic items were also left alone: a typo in 15.2 ("businsses") and the setup time for LocalPipe, which the guide gives as "Minutes" and the site's table as "Seconds".

## 4. Checked against the site and consistent

No change needed.

- Find rates: 75% owner name, 60% combined email, 20% owner phone
- Sub 1% bounce, triple verified, pay on find
- Cross-referencing sources: Google Maps, business websites, Facebook, Instagram, BBB, Secretary of State filings
- Live lookups, not a stored database
- Competitor rows in 13.2: Apollo about 20% / 10% / 7% with 45 minutes of setup, assembled Clay workflow about 30% / 14% / 10% with 6+ hours
- Sourcing cost of roughly a dollar per thousand businesses (6.3) against the site's $1 per 1,000 on Usage-Based
- About 4,000 Maps categories (the published category file has 4,284)
- The API description in 6.8: batched search with exact category matching and filters, enrichment with title waterfalling and a generic-email fallback flag
- Strongest in the United States
- Openmart and Outscraper named as the honest alternatives, Apollo for corporate lists
- 34,700,000 emails sent on LocalPipe data

## 5. On the site but not in the guide

All of this now lives in [`localpipe/`](../localpipe/README.md).

- The plans and prices: Starter $57, Growth $97, Pro $157, and Usage-Based at $97 plus per-find pricing
- Credit costs: business email 1, owner email 2, owner phone 4, owner names included
- Annual billing at 40% off, with a year of credits upfront
- Unlimited credit rollover and unlimited seats
- No double charge for a contact found in the last 30 days
- CSV upload of an existing list
- The Data Recovery panel
- Custom title waterfalling raising hard-to-find roles from roughly 50% to 75%
- The October 2026 benchmark: 51.6% verified sendable coverage against Outscraper 12.0%, Seamless 7.0%, Apollo 5.2%, Hunter 4.5%
- Confirmed international coverage in Australia, the UK, Germany, Canada and the United States
- 100% of Google Maps, roughly 250 million businesses by third-party estimate
- Published competitor pricing for Openmart, Outscraper, Clay, Apollo, ZoomInfo and Lusha

## 6. Structural changes in the conversion

Nothing below changes what the guide says.

- **Two editions merged.** The Agency Guide and the Business Owner Guide share 83 pages. Each is stored once, with the Business Owner text as the body. Of those, 33 are identical in both editions and 50 carry a closing "Agency edition differences" section listing the passages that read differently. Chapters 14 and 16 differ too much to merge and have one file per edition. 14 pages are agency only and 13 are business-owner only.
- **Promotional blocks moved.** The LocalPipe and Legacy GTM offer blocks that closed most chapters were lifted into [reference/cta-library.md](../reference/cta-library.md): 23 distinct LocalPipe blocks and 23 distinct Legacy GTM blocks, each with the chapters it closed. Chapter 16 keeps its offers in place because they are its content.
- **Callouts.** Notion `<aside>` blocks became Markdown blockquotes with their icon.
- **Citation tags.** Eighteen stray `<cite>` tags left by the export were removed. The text inside them was kept.
- **Images.** Two headshots on the front page were dropped. They remain in the archived export.
- **Headings.** One second top-level heading on the front page was demoted so each file has a single title. Where the two editions capitalised a title differently, the Title Case version was used.
- **Links.** All 394 internal links were rewritten to the new paths. External links are unchanged.
- **File names.** Notion page IDs, spaces and special characters were removed. Chapter numbers are kept.
- **Front matter.** Each file gained a short header: title, chapter, section, editions, a key idea taken from the chapter's opening callout, and the source.

**Verification.** After conversion, all 6,703 text blocks in the export were checked against the repo. None are missing. The three corrected passages are accounted for by their original wording, recorded in section 1.
