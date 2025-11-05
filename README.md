# Federated Trends Discovery - Misskey Play App Suite

A modular collection of AiScript-based Play apps for Misskey/Sharkey that display trending content from your local instance, without requiring any external aggregators or proxies.

## 📦 Modular Architecture (v0.9+)

The suite is now split into **three independent Play apps** for better maintainability and performance:

### 1. **Hashtags Trending** (`hashtags-trending.play.aiscript`)
- 📊 Display trending hashtags from local instance
- 🎯 Click tags to view recent posts
- 💾 Snapshot caching (up to 12 snapshots)
- 🔝 Top 5 tags highlighted
- ↕️ Sorted by popularity

### 2. **Media Thumbnails** (`media-thumbnails.play.aiscript`)
- 🖼️ Display recent media posts (images)
- 🎥 Display video posts
- 👤 Show author usernames
- 🔄 Manual refresh

### 3. **Posts Carousel** (`posts-carousel.play.aiscript`)
- 📝 Browse trending posts one at a time
- ◀️▶️ Previous/Next navigation
- 👤 Author information
- 🔄 Manual refresh

## Features

### ✅ Implemented (v0.9)

- **📊 Trending Hashtag Cloud**: Displays trending hashtags with font-size scaling based on popularity
- **🎬 Trending Media Wall**: Shows trending images and videos from popular posts
- **💾 Snapshot Caching**: Stores trend history locally using `Mk:save` for persistent data
- **🔄 Manual Refresh**: Refresh trends on-demand with a single click
- **🎯 Interactive Tags**: Click hashtags to view sample posts
- **📱 Responsive Design**: Works on mobile and desktop viewports
- **🌐 Federated Discovery**: Detects mutual servers (foundation for future federation features)

### 🚧 Limitations

- **Manual Refresh Only**: Due to AiScript Play limitations, automatic periodic refresh is not available
- **Local Trends Only (v1.0)**: This version focuses on local instance trends; mutual server trend aggregation is prepared but not active
- **No CORS Workaround**: External server probing may be blocked by CORS policies

## Installation

### Quick Start: Install One or All Apps

You can install any combination of the three apps. Each works independently!

#### App 1: Hashtags Trending (Recommended Start)

1. Navigate to your Misskey/Sharkey Play section (`/play`)
2. Click **"Create New Play"**
3. **Copy** the contents of [`hashtags-trending.play.aiscript`](./hashtags-trending.play.aiscript)
4. **Paste** into the Play editor
5. **Name**: "Hashtags Trending"
6. **Description**: "Display and explore trending hashtags"
7. Click **"Save"** and **"Play"**

#### App 2: Media Thumbnails (Optional)

1. Create another new Play
2. **Copy** the contents of [`media-thumbnails.play.aiscript`](./media-thumbnails.play.aiscript)
3. **Paste** into the Play editor
4. **Name**: "Trending Media"
5. **Description**: "Browse recent media posts"
6. Click **"Save"** and **"Play"**

#### App 3: Posts Carousel (Optional)

1. Create another new Play
2. **Copy** the contents of [`posts-carousel.play.aiscript`](./posts-carousel.play.aiscript)
3. **Paste** into the Play editor
4. **Name**: "Posts Carousel"
5. **Description**: "Browse trending posts one by one"
6. Click **"Save"** and **"Play"**

### Why Modular?

- ✅ **Easier to Debug**: Each app is simpler and self-contained
- ✅ **Better Performance**: Run only what you need
- ✅ **Independent Testing**: Test features separately
- ✅ **Gradual Rollout**: Install hashtags first, add others later

## Usage

### Main Interface

The app displays two main sections:

#### 1. Trending Hashtags

- Hashtags are displayed with varying font sizes based on popularity
- Larger text = more popular hashtag
- Click any hashtag to view recent posts using that tag

#### 2. Trending Media

- Shows media (images/videos) from trending posts
- Click media items to view post details
- Limited to 25 items for performance

### Controls

- **🔄 Refresh Trends**: Manually fetch latest trending data
- **Status Bar**: Shows active sources, tag count, and media count

### Data Persistence

- Trend snapshots are cached locally using `Mk:save`
- Up to 12 snapshots retained (approximately 2 hours of history)
- Data persists across browser sessions

## Configuration

You can modify behavior by editing constants in the `CONFIG` object:

```aiscript
let CONFIG = {
  MAX_SNAPSHOTS: 12,           // Maximum cached snapshots
  MAX_TAGS_DISPLAY: 30,        // Maximum hashtags to display
  MAX_MEDIA_ITEMS: 25,         // Maximum media items to show
  MAX_VIDEO_EMBEDS: 3,         // Maximum video embeds
  FONT_MIN: 14,                // Minimum hashtag font size (px)
  FONT_MAX: 36,                // Maximum hashtag font size (px)
  FONT_SCALE: 22,              // Font scaling factor
  TAG_FETCH_LIMIT: 25,         // Tags fetched from API
  POST_SAMPLE_LIMIT: 3         // Posts sampled per tag
}
```

## Technical Architecture

### Data Flow

```
Browser (Play App)
    ↓
Local Instance API
    ├── Mk:api('hashtags/trend')
    ├── Mk:api('notes/search-by-tag')
    └── Mk:api('users/following')
    ↓
Local Storage (Mk:save)
    └── trends_snapshots
```

### API Endpoints Used

| Endpoint | Purpose | Fallback |
|----------|---------|----------|
| `hashtags/trend` | Fetch trending tags | None |
| `notes/search-by-tag` | Get posts for a tag | `notes/search` |
| `users/following` | Detect mutual servers | `i/following` |

### Data Structures

**Snapshot Storage** (`Mk:save("trends_snapshots")`):
```json
{
  "snapshots": [
    {
      "timestamp": 1699000000000,
      "sources": {
        "local": {
          "tags": {"fediverse": 120, "cats": 40}
        }
      }
    }
  ]
}
```

## Performance Considerations

- **Tag Limit**: Maximum 30 tags displayed to prevent UI clutter
- **Media Limit**: Maximum 25 media items for rendering performance
- **Snapshot Limit**: Maximum 12 snapshots to limit storage usage
- **Lazy Loading**: Media items are loaded on-demand (where supported)

## Troubleshooting

### No Trends Appearing

1. **Check Instance Support**: Ensure your instance has the `hashtags/trend` API endpoint
2. **Verify Activity**: Instance must have recent hashtag activity
3. **Try Refresh**: Click the "🔄 Refresh Trends" button

### "Local trends unavailable" Error

- Your instance may not support trending hashtags
- Check instance version (Misskey v12.119+ or Sharkey v2023.11+)
- Verify API endpoint at `https://your-instance.tld/api-doc`

### No Media Showing

- Trending posts may not contain media attachments
- Try refreshing after some time
- Check if posts in the trending tags actually have images/videos

### CORS Errors (Console)

- Expected behavior for mutual server probing
- Does not affect local trend functionality
- Mutual server features reserved for future updates

## Development

### Project Structure

```
Sharkey-Trending-App/
├── federated-trends-discovery.play.aiscript   # Main app
├── README.md                                   # This file
├── SPEC.md                                     # Original specification
└── CHANGELOG.md                                # Version history
```

### Contributing

This is an implementation of SPEC-001. To contribute:

1. Fork the repository
2. Create a feature branch
3. Test thoroughly on a Misskey/Sharkey instance
4. Submit a pull request with detailed description

### Testing Checklist

- [ ] App loads without errors
- [ ] Trending hashtags appear and are clickable
- [ ] Font sizes vary based on popularity
- [ ] Media items display correctly
- [ ] Refresh button updates data
- [ ] Data persists after browser refresh
- [ ] Mobile viewport displays correctly
- [ ] No console errors (except expected CORS from mutual probes)

## Roadmap

### Version 1.x
- [x] Local trending hashtags
- [x] Hashtag cloud with font scaling
- [x] Media wall for trending content
- [x] Snapshot caching
- [ ] Enhanced error messaging
- [ ] Custom CSS themes

### Version 2.x (Future)
- [ ] Active mutual server trend aggregation
- [ ] Weighted source blending
- [ ] Time window selection UI
- [ ] List view mode toggle
- [ ] Export trends data

### Version 3.x (Future)
- [ ] Advanced filtering options
- [ ] Hashtag history charts
- [ ] Custom refresh intervals
- [ ] Multi-instance comparison

## Specification Compliance

This implementation follows **SPEC-001-Misskey Play — Federated Content Discovery (Final AiScript-Only Design)**.

### Requirements Status

| Requirement | Status | Notes |
|-------------|--------|-------|
| Display trending hashtags | ✅ Complete | Local instance only |
| Clickable hashtags | ✅ Complete | Shows sample posts |
| Font-size scaling | ✅ Complete | 14-36px range |
| Media wall display | ✅ Complete | Images + videos |
| Cache with Mk:save | ✅ Complete | 12 snapshot limit |
| Responsive layout | ✅ Complete | Mobile-friendly |
| Mutual server detection | ⚠️ Partial | Detected but not actively used |
| Video embeds | ⚠️ Limited | Basic support, limited by AiScript UI |

## License

This project is open source and available for use, modification, and distribution.

## Credits

- **Specification**: SPEC-001-Misskey Play
- **Implementation**: AiScript for Misskey/Sharkey
- **Platform**: Misskey/Sharkey federated social network

## Support

For issues, questions, or feature requests:
1. Check this README and troubleshooting section
2. Review the SPEC.md for architectural details
3. Test on latest Misskey/Sharkey version
4. Report issues with detailed reproduction steps

---

**Version**: 1.0.0
**Last Updated**: 2025-11-05
**AiScript Version**: 0.18.0
**Misskey Compatibility**: v12.119+, Sharkey v2023.11+
