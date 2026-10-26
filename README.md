# Apify (apify)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Apify is a full-stack web scraping and browser automation platform that enables developers to build, run, and scale web scrapers, crawlers, and data extraction tools using a cloud-based infrastructure with built-in proxy management, scheduling, and storage. The platform hosts thousands of ready-made Actors for scraping social media, search engines, maps, e-commerce sites, and more.

**URL:** [https://apify.com](https://apify.com)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - Actors, Browser Automation, Crawling, Data Aggregation, Data Extraction, Web Automation, Web Scraping

## Timestamps

- **Created:** 2026-03-26
- **Modified:** 2026-04-19

## APIs

### Apify API
The Apify REST API (v2) provides programmatic access to the Apify platform, allowing you to manage actors, run scraping tasks, access datasets, key-value stores, and request queues.

**Human URL:** [https://apify.com](https://apify.com)

#### Tags:

 - Actors, Automation, Crawling, Data Extraction, Datasets, Web Scraping

#### Properties

- [Documentation](https://docs.apify.com/api/v2)
- [OpenAPI](openapi/apify-api.yaml)
- [GettingStarted](https://docs.apify.com/api/v2/getting-started)
- [Authentication](https://docs.apify.com/api/v2#authentication)
- [RateLimits](https://docs.apify.com/api/v2#rate-limiting-and-scaling)
- [JSONSchema](json-schema/apify-actor-schema.json)
- [JSONSchema](json-schema/apify-run-schema.json)
- [JSONSchema](json-schema/apify-dataset-schema.json)
- [JSONSchema](json-schema/apify-key-value-store-schema.json)
- [JSON-LD](json-ld/apify-context.jsonld)
- [Node.js Client SDK](https://www.npmjs.com/package/apify-client)
- [Python Client SDK](https://pypi.org/project/apify-client/)

## Common Properties

- [Website](https://apify.com)
- [Documentation](https://docs.apify.com)
- [GettingStarted](https://docs.apify.com/api/v2/getting-started)
- [Pricing](https://apify.com/pricing)
- [Blog](https://blog.apify.com)
- [SignUp](https://console.apify.com/sign-up)
- [Login](https://console.apify.com/sign-in)
- [Academy](https://docs.apify.com/academy)
- [Support](https://help.apify.com)
- [GitHubOrganization](https://github.com/apify)
- [CLI](https://www.npmjs.com/package/apify-cli)

## Features

| Name | Description |
|------|-------------|
| Actors Marketplace | Store of thousands of pre-built web scrapers and automation tools ready to run with zero configuration. |
| Cloud Infrastructure | Run Actors on Apify's scalable cloud infrastructure with built-in proxy rotation, scheduling, and storage. |
| Datasets | Structured storage for Actor output with multi-format export (JSON, CSV, XML, XLSX, etc.). |
| Key-Value Stores | Persistent key-value storage for arbitrary data including files, screenshots, and configuration. |
| Request Queues | URL queue management for large-scale distributed web crawling. |
| Proxy Management | Built-in datacenter and residential proxy pools with automatic rotation. |
| Scheduled Runs | Schedule Actors to run automatically on cron schedules. |
| MCP Server | Apify MCP server enabling AI agents to use thousands of web scraping and automation tools. |

## Use Cases

| Name | Description |
|------|-------------|
| AI Training Data Collection | Extract structured data from websites for LLM training datasets, RAG pipelines, and AI applications. |
| E-commerce Price Monitoring | Scrape product prices, availability, and reviews from e-commerce websites for competitive intelligence. |
| Social Media Data Extraction | Extract posts, profiles, and engagement data from social media platforms. |
| Search Engine Data | Scrape search engine results, SERP data, and web listings for SEO and market research. |
| Lead Generation | Extract business data from directories, LinkedIn, and other professional platforms. |

## Integrations

| Name | Description |
|------|-------------|
| Crawlee | Open-source web crawling library for Node.js and Python built by Apify. |
| Zapier | Zapier integration for connecting Apify Actors with 5000+ apps. |
| Make (Integromat) | No-code automation platform integration for Apify. |
| LangChain | LangChain integration for using Apify data loaders in AI applications. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Apify API](openapi/apify-api.yaml)

### JSON Schema

- [apify-actor-schema.json](json-schema/apify-actor-schema.json)
- [apify-run-schema.json](json-schema/apify-run-schema.json)
- [apify-dataset-schema.json](json-schema/apify-dataset-schema.json)
- [apify-key-value-store-schema.json](json-schema/apify-key-value-store-schema.json)

### JSON Structure

- [apify-actor-structure.json](json-structure/apify-actor-structure.json)
- [apify-run-structure.json](json-structure/apify-run-structure.json)
- [apify-dataset-structure.json](json-structure/apify-dataset-structure.json)
- [apify-key-value-store-structure.json](json-structure/apify-key-value-store-structure.json)

### JSON-LD

- [apify-context.jsonld](json-ld/apify-context.jsonld)

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Apify API](capabilities/shared/apify.yaml) — 8 operations for Actors, runs, and datasets

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Web Scraping and Automation](capabilities/web-scraping-automation.yaml) | Apify API | 5 | Data Engineer, AI Developer, Web Scraping Engineer |

## Vocabulary

- [Apify Vocabulary](vocabulary/apify-vocabulary.yaml) — Unified taxonomy mapping 6 resources, 4 actions, 1 workflow, and 3 personas across operational (OpenAPI) and capability (Naftiko) dimensions

## Rules

- [apify-spectral-rules.yml](rules/apify-spectral-rules.yml) — 11 rules across 6 categories enforcing Apify API conventions

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
