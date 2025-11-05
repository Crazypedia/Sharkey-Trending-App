# SPEC-001-Misskey Play — Federated Content Discovery (Final AiScript-Only Design)

## Background

Local Sharkey/Misskey instances often lack rich discovery features. Users want an AiScript Play app that visualizes local and mutual-instance trending hashtags and posts, without external servers. This version focuses on a static hashtag cloud with clickable hashtags and a lightweight combined media wall for images and PeerTube videos.

## Requirements

**Must (M)**

* Display trending hashtags from local instance and mutual servers that expose public trend APIs.
* Implement clickable hashtags that show representative posts.
* Use font-size scaling to represent relative popularity in a static hashtag cloud.
* Display trending images and up to three PeerTube video embeds in a combined media wall.
* Cache fetched data in `Mk:save` and display time options based on collected snapshots.
* Maintain legibility and browser performance.

**Should (S)**

* Ensure consistent color contrast and font-size bounds to preserve readability.
* Use responsive DOM layout that adapts to narrow and wide viewports.
* Provide warning banners for high-resource operations (e.g., enabling multiple mutual servers).

**Could (C)**

* Allow users to toggle between cloud, media wall, and list modes.

**Won't (W)**

* Use any external aggregators (e.g., Fedi.buzz) or proxies.
* Provide 1-day persistent trends.

## Method

### Hashtag Cloud Generation

* Hashtags displayed as inline `<span>` elements inside a flex-wrap container.
* Font size proportional to normalized popularity: `fontSize = minFont + (score / maxScore) * scale`.
* Limit max/min size ratio (e.g., 1.2–2.5× base size) for readability.
* Set font-weight proportional to score to enhance emphasis.
* Apply contrasting text/background colors using theme-aware CSS variables (`var(--accentColor)`).
* Sort hashtags alphabetically or by score for predictable layout.
* Clicking a hashtag triggers a post view fetch for that tag.

**Legibility Considerations:**

* Use word-break: keep hashtags on single line, avoid wrap inside tag.
* Prevent overlapping text via flex gaps and min font size (≥12 px).
* Limit total displayed tags to 25–30 for clarity.

### Media Wall (Images + Videos)

* Display image and video thumbnails in a responsive grid.
* Use CSS grid: `grid-template-columns: repeat(auto-fit, minmax(120px, 1fr))`.
* For images: use `note.files[].thumbnailUrl` when available.
* For PeerTube videos: detect `video/*` attachments or PeerTube domain URLs.
* Up to 3 videos embedded as `<iframe src=".../embed" loading="lazy">`.
* If embed fails (CORS/CSP), fall back to thumbnail preview image linking to PeerTube.
* Images and video thumbnails open modal post preview on click.

**Performance Notes:**

* Lazy load all media.
* Cap total media items (images + videos) ≤ 25.
* Reuse cached posts to avoid redundant API calls.

### UI Layout Example

```
<div class="trends">
  <div class="hashtag-cloud">
    <span style="font-size:18px">#fediverse</span>
    <span style="font-size:14px">#misskey</span>
    ...
  </div>
  <div class="media-wall">
    <img src="thumb1.jpg" loading="lazy" />
    <iframe src="https://peertube.example/embed/abc" loading="lazy"></iframe>
    ...
  </div>
</div>
```

### Snapshot Storage and Dynamic Time Periods

* Each fetch saves `{timestamp, tagScores}` in `Mk:save('snapshots')`.
* Limit 12 snapshots (≈2 h history).
* Display relative periods ("Past 10 m", "Past 1 h", "Since open") based on timestamps.

### PlantUML Summary

```plantuml
@startuml
node "Browser (Play App)" as Browser
node "Local Instance (Sharkey)" as Local
node "Mutual Servers" as Mutual

Browser --> Local : Mk:api('hashtags/trend')
Browser --> Local : Mk:api('i/following')
Browser --> Mutual : probe /api/hashtags/trend
Browser : render hashtag cloud, media wall
Browser : store snapshots in Mk:save
@enduml
```

## Implementation

1. **Initialization**

   * Fetch local trends and identify mutual servers with accessible trend endpoints.
   * Load cached snapshots and compute available relative time periods.

2. **UI Rendering**

   * Render hashtag cloud first; apply scaled fonts and click handlers.
   * Render combined media wall (thumbnails, limited video embeds).
   * Show warnings dynamically for resource-intensive settings.

3. **Refresh Cycle**

   * Fetch new data every 10 minutes; update cached snapshots.
   * Recompute normalization and refresh hashtag cloud and media grid.

## Milestones

1. Static hashtag cloud and post view — 1 week.
2. Combined image/video media wall — 1 week.
3. Caching, snapshots, and time period logic — 1 week.
4. UI refinement, warnings, and performance validation — 1 week.

## Gathering Results

* Verify readable cloud layout on multiple screen sizes.
* Validate correct scaling of font sizes vs scores.
* Confirm that image and video displays stay within resource budget.
* Measure refresh latency and memory footprint.

---

## Technical Additions for LLM-driven Implementation

These additions supply concrete schemas, API calls, UI structure, CSS guidance, error handling, and AiScript pseudocode so an LLM can generate a workable Misskey Play app.

### Data Structures / JSON Schemas

**Cached snapshot (stored in Mk:save under key `trends_snapshots`)**

```json
{
  "snapshots": [
    {
      "timestamp": 1699000000000,
      "sources": {
        "local": {
          "tags": {"fediverse": 120, "cats": 40}
        },
        "mutual:example.com": {
          "tags": {"fedi": 60, "fediverse": 30}
        }
      }
    }
  ]
}
```

**Canonical tag object (in-memory)**

```json
{
  "tag": "fediverse",
  "scores": {"local":120, "mutual:example.com":30},
  "normalizedScore": 0.87,
  "samplePosts": [ {"id":"noteId","url":"...","text":"...","files":[...] } ]
}
```

**Canonical post object**

```json
{
  "id": "noteId",
  "url": "https://instance/notes/noteId",
  "author": {"id":"usrId","username":"alice","host":"example.com"},
  "text": "Post text...",
  "files": [ {"type":"image","thumbnailUrl":"...","url":"..."}, {"type":"video","site":"peertube","url":"...","thumbnailUrl":"..."} ]
}
```

### API Endpoints and Parameters

**Local trends (recommended call via AiScript helper):**

* `Mk:api('hashtags/trend', {limit: 25})` — returns local trending tags and metadata.

**Local following to detect mutual servers:**

* `Mk:api('i/following', {limit: 100})` — examine `acct` fields of returned profiles to extract hostnames.

**Mutual server probe (client fetch):**

* Probe `https://{host}/api/hashtags/trend` (Misskey). If 405/404 try `https://{host}/api/v1/trends/tags` (Mastodon).
* Use `fetch()` and test for CORS errors. If `fetch` fails due to CORS, mark as unavailable.

**Fetch posts for a tag**

* If source is Misskey-like: `POST https://{host}/api/notes/search` with body `{q:"#tag", limit:3}` or `GET /api/notes/timeline/tag/{tag}` if available.
* If Mastodon-like: `GET https://{host}/api/v1/timelines/tag/{tag}?limit=3`.
* Prefer local calls via `Mk:api` for local tags to avoid CORS.

### UI Component Hierarchy (DOM / Reactive Mapping)

1. `div.trends-root`

   * `header.controls`

     * `select#timeWindow` (dynamic options)
     * `div.source-controls` (local + mutual toggles and weight dropdowns)
     * `button#refresh`
   * `section.hashtag-cloud` (grid/flex container)

     * multiple `button.tag` elements (data-tag attr)
   * `section.media-wall` (grid)

     * `div.media-item` (img or iframe)
   * `div.modal.preview` (hidden by default)

### CSS Class Names and Responsive Breakpoints

* `.trends-root { padding: 8px; font-family: system-ui; }`
* `.hashtag-cloud { display:flex; flex-wrap:wrap; gap:8px; align-items:center; }`
* `.tag { background:var(--backgroundAccent); padding:6px 10px; border-radius:999px; cursor:pointer; white-space:nowrap; }`
* `.media-wall { display:grid; grid-gap:8px; grid-template-columns: repeat(auto-fit, minmax(120px, 1fr)); }
* `.media-item img, .media-item iframe { width:100%; height:auto; display:block; }
* **Breakpoints:**

  * `@media (max-width:600px)` reduce base font-size by 10% and set grid minmax to 100px.
  * `@media (min-width:1200px)` increase grid columns and allow 4+ columns.

**Accessibility:**

* Ensure `button.tag` has `aria-label` with tag and score. Provide keyboard focus styles.

### Error Handling and Fallbacks

* **If Mk:api('hashtags/trend') fails:** show fallback message "Local trends unavailable" and allow user to retry.
* **If mutual probe fails for a host:** mark host disabled and show tooltip "Trend endpoint not accessible (CORS or unavailable)."
* **If fetching posts for a tag fails:** show "No sample posts available" under the tag.
* **If media embed blocked:** show thumbnail with link to external content.
* **Rate limiting:** respect server `Retry-After` if presented; otherwise implement exponential backoff per host.

### AiScript Pseudocode and Function Outlines

Below is a detailed pseudocode sketch designed to be directly translated into AiScript (misskey Play) code.

```
// Initialization
async function init() {
  ui.showLoading()
  const cached = await Mk.load('trends_snapshots') || {snapshots:[]}
  state.snapshots = cached.snapshots
  await fetchLocalTrends()
  await detectMutualServers()
  ui.renderControls()
  ui.renderCloud()
  ui.renderMediaWall()
  ui.hideLoading()
  schedulePeriodicRefresh()
}

async function fetchLocalTrends() {
  try {
    const res = await Mk.api('hashtags/trend', {limit:25})
    state.localTags = normalizeMisskeyTrendResponse(res)
    saveSnapshot('local', state.localTags)
  } catch (e) {
    ui.showError('Local trends fetch failed')
  }
}

async function detectMutualServers() {
  const following = await Mk.api('i/following', {limit:100})
  const hosts = extractUniqueHosts(following)
  for (host of hosts.slice(0,8)) { // limit probe set
    const ok = await probeHostForTrends(host)
    if (ok) state.mutualCandidates.push(host)
  }
}

async function probeHostForTrends(host) {
  try {
    // attempt misskey endpoint
    const res = await fetch(`https://${host}/api/hashtags/trend`, {method:'GET'})
    if (res.ok) { return true }
    // try mastodon trends
    const res2 = await fetch(`https://${host}/api/v1/trends/tags`, {method:'GET'})
    if (res2.ok) { return true }
    return false
  } catch (err) {
    // likely CORS or network error
    return false
  }
}

function saveSnapshot(sourceName, tagsMap) {
  const now = Date.now()
  state.snapshots.push({timestamp: now, sources: {[sourceName]: {tags: tagsMap}}})
  // trim to last 12
  if (state.snapshots.length > 12) state.snapshots.shift()
  Mk.save('trends_snapshots', {snapshots: state.snapshots})
}

function computeNormalizedScores() {
  // aggregate scores from available sources for displayed tags
}

function renderCloud() {
  const tags = computeTopTags()
  const maxScore = Math.max(...tags.map(t=>t.normalizedScore))
  for (t of tags.slice(0,30)) {
    const size = minFont + (t.normalizedScore / maxScore) * scale
    ui.createTagElement(t.tag, size, t)
  }
}

function renderMediaWall() {
  const mediaPosts = collectMediaPosts(limit=25)
  // ensure at most 3 video embeds
  ui.renderMediaGrid(mediaPosts)
}

function onTagClick(tag) {
  // fetch posts for tag from highest priority source available
  // render post preview modal
}

function schedulePeriodicRefresh() {
  setInterval(async ()=>{
    await fetchLocalTrends()
    // optionally fetch mutuals if enabled
    ui.refreshAll()
  }, 10*60*1000)
}
```

### Implementation Handoff Checklist for LLM

* Implement `Mk.api` use for local endpoints and `fetch()` for mutual probes with CORS detection.
* Provide DOM templates for `.tag`, `.media-item`, and modal preview.
* Implement lightweight CSS per the classes above and responsive breakpoints.
* Implement snapshot persistence using `Mk.save` and `Mk.load` with trimming policy.
* Add logging and graceful error UI for failures.

---

## Implementation Status (v1.0.0)

### ✅ Completed

- [x] Display trending hashtags from local instance
- [x] Clickable hashtags showing representative posts
- [x] Font-size scaling (14-36px) based on popularity
- [x] Font-weight scaling (400-700) for emphasis
- [x] Display trending media (images + videos)
- [x] Cache with `Mk:save` (12 snapshot limit)
- [x] Maintain legibility and performance
- [x] Responsive layout for mobile/desktop
- [x] Error handling and fallbacks
- [x] Mutual server detection (foundation)

### ⚠️ Partial / Limited

- [⚠️] PeerTube video embeds (limited by AiScript UI capabilities)
- [⚠️] Mutual server trend aggregation (detected but not active due to CORS)
- [⚠️] Time window selection (snapshots stored but UI not implemented)
- [⚠️] Periodic refresh (manual only due to AiScript limitations)

### 🚧 Future Work

- [ ] Active mutual server federation
- [ ] Time window selector UI
- [ ] View mode toggles (cloud/list/grid)
- [ ] Warning banners for high-resource operations
- [ ] Advanced CSS theme support
- [ ] Automatic refresh (if AiScript adds support)

---

**Specification Version**: 1.0
**Implementation Version**: 1.0.0
**Last Updated**: 2025-11-05
