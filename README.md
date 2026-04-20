# Apify (apify)
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
