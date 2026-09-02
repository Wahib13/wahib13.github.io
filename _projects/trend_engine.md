---
title: Trend Engine
order: 2
permalink: /projects/trend_engine.html
summary: A personal news aggregator with semantic search. Pulls articles from RSS feeds, embeds and keyword-tags them, and surfaces what is being talked about across sources without you having to scroll the feeds yourself.
stack: [Python, FastAPI, PostgreSQL, pgvector, sentence-transformers, KeyBERT, React, TypeScript, Docker]
status: Running at app1.wahib.dev
image: /assets/images/project-mockup-trend-engine.jpeg
live_link: 'https://app1.wahib.dev'
project_link: 'https://github.com/Wahib13/trend-engine/'
---

## Why

I wanted a single page that told me what the news was about today, grouped by topic, without reading every headline from every source. Existing aggregators either showed me everything or picked for me. This one shows the shape of the day and lets me drill in.

## How it works

The backend is a pipeline that runs on a schedule:

1. **Fetch.** Pull entries from a small set of RSS feeds (currently BBC and The Guardian, across Politics, Technology, Business and Health).
2. **Download.** Fetch the full article text rather than trusting the feed excerpt.
3. **Embed.** Generate a 384-dimensional sentence embedding per article and store it in Postgres using pgvector.
4. **Extract.** Pull keywords from titles with KeyBERT. The feed's own topic is always kept as a first-class category so grouping stays consistent even when keywords wander.
5. **Summarise.** Produce a per-topic summary so the front page reads as a short briefing rather than a list of links.

A FastAPI service exposes the results over REST. The React front end groups articles by keyword, filters by source, and supports semantic search over the embeddings. I am currently extending the pipeline into an event-driven, real-time architecture with WebSocket delivery and live keyword extraction as articles arrive.

## What I learned

- pgvector is good enough for this scale and keeps everything in one database, which is worth a lot operationally.
- Keyword extraction on titles alone is noisier than on bodies, but far cheaper, and the topic-as-category fallback covers most of the gap.
- The whole thing runs from one `docker compose up`, which is how I want every side project to behave.
