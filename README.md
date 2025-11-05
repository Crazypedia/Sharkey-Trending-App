# Federated Trends Discovery - Misskey Play App

A comprehensive AiScript-based Play app for Misskey/Sharkey that displays trending hashtags and media from your local instance and federated mutual servers, without requiring any external aggregators or proxies.

## Features

### ✅ Implemented (v1.0)

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

### Step 1: Open Misskey/Sharkey Play

1. Navigate to your Misskey/Sharkey instance
2. Go to **"Play"** section (usually at `/play`)
3. Click **"Create New Play"**

### Step 2: Copy the Code

1. Open [`federated-trends-discovery.play.aiscript`](./federated-trends-discovery.play.aiscript)
2. Copy the entire contents
3. Paste into the Play editor

### Step 3: Configure & Publish

1. Give your Play a name: **"Federated Trends Discovery"**
2. Add a description: **"Discover trending hashtags and media from your instance"**
3. Set visibility (Public/Home/Followers)
4. Click **"Save"** or **"Publish"**

### Step 4: Run the App

1. Click **"Play"** to execute
2. Wait for initial trends to load
3. Interact with hashtags and media items

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
