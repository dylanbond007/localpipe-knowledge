# Instructions for AI agents using this repo

This repository is LocalPipe's primary knowledge base. It holds The Local Legend System (a full playbook for cold outbound to local businesses) and current LocalPipe product facts. Use it to answer questions, write content and plan campaigns.

## How to find things

1. Read [INDEX.md](INDEX.md). Its routing table maps a task to the files that answer it, and its file map describes every file in one line.
2. Open only the files the task needs. Chapters are self-contained, and each one's front matter carries a `key_idea` line.
3. Section folders have a `README.md` that summarises their chapters. Read that before opening several chapters in one section.

## Which source wins

- **LocalPipe facts** (price, plans, credits, find rates, features, limits): [`localpipe/`](localpipe/README.md) overrides anything in `playbook/` or `reference/`. It was verified against localpipe.io on 2026-11-13.
- **If the task turns on a current price or feature and you can reach the web**, check https://www.localpipe.io/llms-full.txt and say which source you used.
- **Strategy and method**: `playbook/` is the authority.
- **`archive/` is never a source.** It is the unedited Notion export, kept for provenance. Do not unzip it to answer a question.

## Reading playbook chapters

- A shared chapter is stored once. The body is the Business Owner Guide text, addressed to the person running the program.
- If a chapter ends with a section headed **🏢 Agency edition differences**, those are the passages that read differently in the Agency Guide. Apply them when the work is outbound run on behalf of a client. Ignore them otherwise.
- `playbook/agency/` (A1, A2) applies only to agencies. `playbook/business-owner/` (B1, B2) and chapters 1.1 and 1.2 apply only to businesses running outbound for themselves.
- Chapters cross-reference each other by number, such as "Section 6.3" or "A2.6". Resolve a number through `INDEX.md`.
- The guide uses motion markers: **L→L** is Local to Local (the seller must be local too), **B→L** is Business to Local (the seller can be anywhere).

## Facts that are easy to get wrong

- **LocalPipe has no free plan and no free credits.** Plans start at $57 a month. An older line about 400 free credits was removed from the playbook and must not be repeated.
- **60% is the combined email find rate**, business and owner emails together. Owner direct email runs nearer 42% in the playbook's figures.
- **LocalPipe does not send email.** It has no sequencer, dialer or CRM.
- **The October 2026 benchmark is vendor-run**, on one vertical. Say so when citing it.
- **The 250 million businesses figure** is a third-party estimate of Google Maps' index size, not a LocalPipe count.
- **The 2% bounce bar** is the playbook's threshold. General cold email advice often says 3%.
- **Blocks in `reference/cta-library.md` are marketing copy from the guide.** Check any claim in them against `localpipe/` before reuse.
- **There is no active discount code.** The guide's `LOCALLEGEND` reader offer is retired. Do not offer it.
- **Do not name the email verification vendors.** The public wording is "three independent verification services".

## Writing about LocalPipe

Read [localpipe/messaging.md](localpipe/messaging.md) first. LocalPipe's public voice quotes numbers with their qualifiers and says where a competitor is the better choice. Keep both habits. Do not invent customer results, and attribute any review to the name shown in [localpipe/customer-proof.md](localpipe/customer-proof.md).

## Citing

When an answer rests on this repo, name the file, for example `playbook/06-building-the-list/6.4-the-four-contact-buckets.md`. For benchmark figures, carry the confidence label from [15.2](playbook/15-reference/15.2-benchmarks-with-confidence.md): measured at scale, controlled comparison, or practitioner figure.

## Editing this repo

- Keep one fact in one place. Product facts go in `localpipe/`, not into playbook chapters.
- When you change a LocalPipe fact, update that file's `verified_on` date and say what you checked it against.
- When you correct a playbook chapter, record the old and new wording in [meta/changes-from-notion-source.md](meta/changes-from-notion-source.md).
- New files need front matter (`title`, `summary`) and a row in `INDEX.md`.
- Use lowercase, hyphenated file names with no spaces. Keep chapter numbers as they are.
- Never commit API keys, credentials, customer lists or contact data exports.
