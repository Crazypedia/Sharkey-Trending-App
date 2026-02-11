# Evaluation: Cron Script vs. More Complex Architecture

## What This Repo Currently Is

A suite of 3 AiScript Play apps (~680 lines total) that run entirely inside a Misskey/Sharkey instance's browser. No backend, no server, no database, no build system.

### The three modules

| Module | File | Lines | Status |
|---|---|---|---|
| Hashtag trending | `hashtags-trending.play.aiscript` | 355 | Functional |
| Media thumbnails | `media-thumbnails.play.aiscript` | 172 | Stub |
| Posts carousel | `posts-carousel.play.aiscript` | 154 | Stub |

### API endpoints used

1. `hashtags/trend` — trending hashtags with user counts
2. `notes/global-timeline` — federated posts for additional tag extraction
3. `notes/local-timeline` (with `withFiles: true`) — media posts
4. `notes/local-timeline` — recent posts for carousel
5. `notes/search-by-tag` — sample posts when clicking a hashtag

### Data processing

- Normalize tag scores to 0-1 range (divide by max)
- Merge local + federated tag counts with 2x local weighting
- Bubble sort by score (AiScript has no native sort)
- Filter posts for image/video attachments
- Cache up to 12 snapshots in browser localStorage via `Mk:save`

---

## Assessment: Cron Script Is Sufficient

The data collection and processing is straightforward HTTP API calls with minimal post-processing. A cron script would be **strictly more capable** than the current AiScript implementation.

### Current limitations that a cron script eliminates

| AiScript limitation | Cron advantage |
|---|---|
| No auto-refresh (`setInterval` unsupported in Play) | Runs on schedule by definition |
| CORS blocks multi-instance federation | Server-side requests have no CORS |
| Browser localStorage only (per-device, 5-10MB) | SQLite or JSON files, shared across consumers |
| Data goes stale between manual refreshes | Always fresh within cron interval |
| Cannot download or cache media files | Can fetch and store thumbnails locally |
| Runs only when a user has the Play app open | Runs regardless of browser state |

### What the cron script would do

```
Every 10-15 minutes:
1. POST https://instance/api/hashtags/trend       → trending tags
2. POST https://instance/api/notes/local-timeline  → media posts (withFiles: true)
3. POST https://instance/api/notes/global-timeline → federated tag extraction (optional)
4. Normalize scores, merge, sort
5. Write trends.json to disk (or post summary note back to instance)
```

Estimated size: **100-200 lines of Python or Node**, replacing all three AiScript files, the snapshot system, and the browser dependency entirely.

### What would push beyond a simple cron script

| Goal | Complexity | Recommendation |
|---|---|---|
| Collect trends and post summary back to instance | Cron only | Single script, no server |
| Serve a web dashboard with trends | Cron + static files | Write JSON, serve with nginx |
| Proxy/cache media thumbnails | Cron + small server | Download thumbnails in cron, or add a ~50-line proxy |
| Aggregate from multiple fediverse instances | Cron only | Loop instance URLs, merge results — no CORS issues |
| Real-time streaming updates | Persistent server | WebSocket listener — but the current app doesn't do this either |
| Interactive post browsing with media | Web app | Lightweight frontend reading the JSON output |

---

## Recommendation

**Start with a cron script.** It handles the core use case (trend collection and output) with less code and fewer constraints than the current AiScript approach. The only additions needed depend on the output target:

### Tier 1: Pure cron (simplest)
- Script fetches trends → writes JSON or posts a summary note to the instance
- No persistent process, no web server
- Suitable if the goal is data collection or automated posting

### Tier 2: Cron + static page
- Cron writes `trends.json` to a web-accessible directory
- A single HTML file reads it and renders a tag cloud / media wall
- Served by nginx or any static host
- No application server needed

### Tier 3: Cron + lightweight API
- If other services need to consume the trend data
- Cron writes to SQLite
- Small API server (Flask/Express, ~100 lines) reads on demand
- Still minimal operational complexity

### Not needed
- Docker/container orchestration (overkill for this workload)
- Message queues or job runners (cron is the job runner)
- Full web framework (the data processing doesn't warrant it)
- Database server (SQLite or flat JSON files are sufficient)

---

## Key Observations About the Existing Code

1. **Two of three modules are stubs.** `media-thumbnails` and `posts-carousel` are marked "STUB VERSION - To be enhanced" and contain minimal functionality.

2. **The federation feature doesn't work.** The spec and code have foundation for probing mutual servers, but CORS blocks it in the browser. A server-side script would solve this immediately.

3. **The snapshot system is over-engineered for its constraints.** It stores 12 snapshots in browser localStorage but provides no UI to browse them (noted as "future work" in SPEC.md). A cron script writing timestamped JSON files achieves the same thing with filesystem tooling.

4. **Documentation outweighs code ~2.5:1.** There are ~1,700 lines of markdown documentation for ~680 lines of AiScript. The extensive spec, testing guide, and changelog suggest the project was designed with LLM-assisted development in mind.

5. **AiScript is a significant constraint.** No native sort, no async/await, no modules, no setInterval. These limitations drove implementation decisions (bubble sort, manual refresh, monolithic files) that wouldn't apply in Python or Node.
