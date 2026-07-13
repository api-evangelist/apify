---
title: "Building a multilingual medicine safety Actor on Apify"
url: "https://blog.apify.com/building-a-multilingual-medicine-safety-actor/"
date: "2026-06-30"
author: "Ashutosh Kumar Tiwari"
feed_url: "https://blog.apify.com/rss/"
---
The Medicine Simplifier Actor takes a medicine name in any language and returns patient-friendly safety instructions in the target language, using Lingo.dev for normalization and localization, apify/google-search-scraper for search, and Gemini for summarization constrained to scraped text only. Key-value store caching cut repeat run times from 30-40 seconds to under 10 seconds across seven supported languages.
