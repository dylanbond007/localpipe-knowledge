---
title: "LocalPipe: current product facts"
summary: "Entry point for everything about LocalPipe the product and company. Verified against localpipe.io on 2026-11-13. This folder overrides the playbook wherever the two disagree."
verified_against: "https://www.localpipe.io (home, FAQ, About, data-comparison, compare, best, pricing explainer, llms.txt, llms-full.txt)"
verified_on: "2026-11-13"
---

# LocalPipe: current product facts

This folder is the source of truth for LocalPipe itself: what it does, what it costs, how good the data is, how it compares, and how to build on the API. Every file here was written from the live site on **13 November 2026**.

**Precedence rule.** The playbook in [`../playbook/`](../playbook/README.md) was written earlier and mentions LocalPipe throughout. If a playbook page and this folder disagree about a LocalPipe fact, this folder is right. If this folder and the live site disagree, the live site is right and this folder needs updating.

## One-paragraph summary

LocalPipe (https://www.localpipe.io) is a local business owner data platform for agencies and sales teams that sell to local businesses. You search any business type in any location on Google Maps, export the list, and enrich each business with the owner's name, owner email, business email and direct owner phone, or upload your own CSV. Lookups run live across 11+ sources, every email is triple verified, and you are only charged when a result is found. Average find rates on local, owner-operated businesses: **75% owner name, 60% combined email, 20% owner phone**, with users reporting **sub 1% bounce**. There is **no free plan**; plans start at **$57 a month**.

## Files

| File | Read it for |
| --- | --- |
| [product.md](product.md) | What LocalPipe is, who it is for, how it works, features, what it does not do, coverage, company details |
| [pricing.md](pricing.md) | Plans, credit costs, annual billing, the Usage-Based plan, cost per contact, plan sizing, refunds, the guide-reader code |
| [data-quality-and-benchmark.md](data-quality-and-benchmark.md) | Find rates, verification, what to expect per 1,000 businesses, and the October 2026 five-provider benchmark in full |
| [competitors.md](competitors.md) | LocalPipe against Apollo, Outscraper, Clay, Openmart, ZoomInfo, Lusha, LeadSwift, Seamless and Hunter, including where each one is the better choice |
| [messaging.md](messaging.md) | Positioning, core lines, proof points, honest limits, and the claims that must carry a qualifier |
| [faq.md](faq.md) | The 20 answers published on the site FAQ |
| [customer-proof.md](customer-proof.md) | Published customer reviews and usage stats |
| [content-index.md](content-index.md) | Every page and blog article on the site, with URLs, for linking and research |
| [api/](api/README.md) | API quick reference, a copy of the full API docs, and the business category list |
| [source-snapshots/](source-snapshots/) | The site's own `llms.txt` and `llms-full.txt` as fetched, for provenance |

## Live sources to re-check

When a task depends on a current price or feature, fetch these rather than trusting a dated copy:

- https://www.localpipe.io/llms-full.txt (the site's own product summary for AI systems, kept in step with the pricing cards)
- https://www.localpipe.io/#pricing
- https://www.localpipe.io/faq
- https://www.localpipe.io/localpipe-api-docs.md
- https://www.localpipe.io/localpipe-business-categories.csv

## What changed against the guide

The Notion guide said LocalPipe had a free tier of 400 credits with no card. That is no longer true. See [`../meta/changes-from-notion-source.md`](../meta/changes-from-notion-source.md) for every correction made and every claim that could not be confirmed.
