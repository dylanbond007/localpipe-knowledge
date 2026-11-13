# LocalPipe Core Knowledge

The central knowledge base for LocalPipe and for cold outbound to local businesses. It is written to be read by Claude, through the GitHub connector or from a local checkout, and by people.

It holds two things:

1. **The playbook.** The Local Legend System, the full guide to cold outbound for local businesses, co-authored by Harvey Le (Legacy GTM) and Dylan Bond (LocalPipe). 115 pages covering strategy, deliverability, list building, offer, copy, launch, replies and channels beyond email.
2. **LocalPipe product facts.** What the product does, pricing, data quality, the October 2026 benchmark, competitor comparisons, messaging, FAQ, customer proof and the API. Verified against localpipe.io on **13 November 2026**.

## Start here

| If you are | Open |
| --- | --- |
| Looking for the file that answers a question | [INDEX.md](INDEX.md) |
| An AI agent about to use this repo | [CLAUDE.md](CLAUDE.md), then [INDEX.md](INDEX.md) |
| After a LocalPipe fact (price, find rate, feature) | [localpipe/](localpipe/README.md) |
| Reading the guide front to back | [playbook/](playbook/README.md) |

## Layout

```
README.md                 this file
CLAUDE.md                 rules for AI agents using the repo
INDEX.md                  task routing table and full file map

playbook/                 The Local Legend System
  README.md               guide front page
  00-start-here/ ... 13-tech-stack-and-costs/, 15-reference/
                          shared chapters, one file each, README.md per section
  14-case-studies-*.md    worked example, one version per edition
  16-webinar-hub-*.md     links, codes and next steps, one version per edition
  agency/                 Agency Guide front page, A1 Winning the Client, A2 Running the Engagement
  business-owner/         Business Owner Guide front page, B1 Before You Start, B2 Running It In-House

localpipe/                current product facts (overrides the playbook on LocalPipe facts)
  product.md  pricing.md  data-quality-and-benchmark.md  competitors.md
  messaging.md  faq.md  customer-proof.md  content-index.md
  api/                    quick reference, full API docs, business category list
  source-snapshots/       the site's llms.txt and llms-full.txt as fetched

reference/cta-library.md  every promotional block from the guide, in one place
meta/changes-from-notion-source.md
                          corrections made, claims not confirmed, structural changes
archive/                  the original Notion export (zip), historical only
```

## How the playbook is organised

The Notion guide came in two editions, an Agency Guide and a Business Owner Guide, that share most of their chapters. Here each shared chapter is stored **once**:

- The body is the Business Owner Guide text, written to the person running the program.
- Where the Agency Guide differs, the differing passages are listed in a closing section headed **🏢 Agency edition differences**, with a note of where each one sits.
- Chapters that exist in one edition only live under `playbook/agency/` or `playbook/business-owner/`, except 1.1 and 1.2, which sit in `01-foundations/` and are marked business-owner only.
- Every file's front matter says which editions it covers.

Chapter text is unchanged from the source apart from the corrections and cuts logged in [meta/changes-from-notion-source.md](meta/changes-from-notion-source.md). The promotional blocks that closed most chapters were moved to [reference/cta-library.md](reference/cta-library.md) so the chapters carry strategy only.

## Which source wins

1. **The live site** (https://www.localpipe.io) on anything about LocalPipe today.
2. **`localpipe/`** in this repo, a dated copy of those facts.
3. **`playbook/`** on strategy and method, and as historical context on LocalPipe.
4. **`archive/`** is never a source. It is the unedited export kept for provenance.

## Using it from another Claude session

With the GitHub connector enabled for this repository, point Claude at it in plain words:

> Use the LocalPipe knowledge repo. Read `INDEX.md` first to find the right files, then answer from them.

For a project that should always have this context, add a line to that project's own `CLAUDE.md` naming this repository and telling Claude to read `INDEX.md` before answering questions about LocalPipe or local outbound.

## Keeping it current

- When a price, plan or feature changes on the site, update the matching file in `localpipe/` and its `verified_on` date. Do not edit the playbook to match unless a chapter states something now false, and log it in `meta/` if you do.
- Add new knowledge as new files, and add a row to `INDEX.md` so it can be found.
- Never commit API keys, customer lists or exports of contact data.

## Provenance

- Playbook source: Notion export of The Local Legend System, received 13 November 2026 (kept in `archive/`).
- LocalPipe facts: localpipe.io pages and published files, fetched 13 November 2026.
- Conversion checked block by block: every passage in the export was carried over. The few passages corrected or cut since are logged in `meta/changes-from-notion-source.md`.

Internal knowledge base. Not for public distribution.
