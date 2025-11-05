# Installation Guide - Federated Trends Discovery

Quick-start guide for installing and using the Federated Trends Discovery Play app on your Misskey or Sharkey instance.

## Prerequisites

### Instance Requirements

- **Misskey**: Version 12.119 or higher
- **Sharkey**: Version 2023.11 or higher
- **Required APIs**:
  - `hashtags/trend` endpoint
  - `notes/search-by-tag` or `notes/search` endpoint
  - `users/following` or `i/following` endpoint

### Browser Requirements

- Modern browser (Chrome 90+, Firefox 88+, Safari 14+, Edge 90+)
- JavaScript enabled
- Cookies/LocalStorage enabled

### Account Requirements

- Active Misskey/Sharkey account
- Permission to create Play apps (usually all users)

## Installation Steps

### Method 1: Copy-Paste (Recommended)

1. **Navigate to Play**
   ```
   https://your-instance.tld/play
   ```

2. **Create New Play**
   - Click the **"+"** or **"Create"** button
   - Or use direct URL: `https://your-instance.tld/play/new`

3. **Configure Metadata**
   - **Title**: `Federated Trends Discovery`
   - **Description**: `Discover trending hashtags and media from your instance`
   - **Visibility**: Choose Public, Home, or Followers

4. **Paste Code**
   - Open [`federated-trends-discovery.play.aiscript`](./federated-trends-discovery.play.aiscript)
   - Select all (Ctrl+A / Cmd+A)
   - Copy (Ctrl+C / Cmd+C)
   - Paste into Play editor (Ctrl+V / Cmd+V)

5. **Save & Publish**
   - Click **"Save"** or **"Publish"**
   - Verify no syntax errors appear

6. **Launch**
   - Click **"Play"** or **"Run"** button
   - Wait for initial load (2-5 seconds)

### Method 2: Import from File

If your instance supports file import:

1. Download `federated-trends-discovery.play.aiscript`
2. Navigate to Play section
3. Look for **"Import"** option
4. Select the downloaded file
5. Review and save

### Method 3: Clone from Repository

If your instance supports GitHub integration:

1. Fork this repository
2. Use Play's **"Import from URL"** feature
3. Provide repository URL
4. Instance will fetch and import

## Post-Installation

### First Launch

After launching, the app will:

1. ✅ Load cached snapshots (if any)
2. ✅ Fetch local trending hashtags
3. ✅ Detect mutual servers
4. ✅ Render hashtag cloud
5. ✅ Collect and display trending media

**Expected Time**: 2-5 seconds

### Verify Installation

Check that you see:

- [ ] "🔄 Refresh Trends" button
- [ ] Status bar showing source/tag/media counts
- [ ] "📊 Trending Hashtags" section
- [ ] Multiple hashtag buttons
- [ ] "🎬 Trending Media" section
- [ ] Media items or "No trending media" message

### Troubleshooting Installation

#### Error: "Syntax Error"

**Cause**: Code not copied completely or corrupted

**Solution**:
1. Clear the editor
2. Re-copy the entire file
3. Ensure no extra characters at start/end
4. Save again

#### Error: "API Not Found"

**Cause**: Instance doesn't support required API

**Solution**:
1. Check instance version (`/about` page)
2. Verify `hashtags/trend` endpoint exists
3. Contact admin if endpoint missing
4. Try on a different instance

#### App Shows Blank Screen

**Cause**: JavaScript error or loading issue

**Solution**:
1. Open browser console (F12)
2. Check for error messages
3. Refresh the page
4. Try different browser
5. Clear browser cache

#### "No trending tags" Message

**Cause**: No recent hashtag activity on instance

**Solution**:
1. Wait for more user activity
2. Post some content with hashtags
3. Try "🔄 Refresh Trends" after a while
4. Verify hashtags are actually trending

## Configuration

### Customize Behavior

Edit the `CONFIG` object in the code:

```aiscript
let CONFIG = {
  MAX_SNAPSHOTS: 12,           // Number of cached snapshots
  MAX_TAGS_DISPLAY: 30,        // Maximum tags to show
  MAX_MEDIA_ITEMS: 25,         // Maximum media items
  FONT_MIN: 14,                // Minimum font size (px)
  FONT_MAX: 36,                // Maximum font size (px)
  FONT_SCALE: 22,              // Font scaling factor
  TAG_FETCH_LIMIT: 25,         // Tags from API
  POST_SAMPLE_LIMIT: 3         // Posts per tag
}
```

**To apply changes**:
1. Edit values in Play editor
2. Save
3. Refresh/restart the Play

### Recommended Settings

**For Small Instances** (< 100 active users):
```aiscript
MAX_TAGS_DISPLAY: 15
MAX_MEDIA_ITEMS: 10
```

**For Large Instances** (1000+ active users):
```aiscript
MAX_TAGS_DISPLAY: 40
MAX_MEDIA_ITEMS: 30
```

**For Low-Power Devices**:
```aiscript
MAX_TAGS_DISPLAY: 20
MAX_MEDIA_ITEMS: 15
FONT_MAX: 28
```

## Usage

### Basic Operations

**Refresh Trends**:
1. Click "🔄 Refresh Trends" button
2. Wait 1-2 seconds for update
3. Tags and media will refresh

**View Tag Posts**:
1. Click any hashtag button
2. Dialog shows recent posts
3. Or opens post creation form

**View Media Details**:
1. Click any media item button
2. Dialog shows post preview
3. Includes author and text

### Advanced Usage

**Check Status**:
- Status bar shows: `Sources: 1 | Tags: 25 | Media: 10`
- **Sources**: Active data sources (local + mutual)
- **Tags**: Currently displayed hashtags
- **Media**: Media items in wall

**Browser Console** (for debugging):
1. Press F12 to open DevTools
2. Check Console tab for logs
3. Look for API errors or warnings

### Best Practices

✅ **DO**:
- Refresh periodically (every 10-15 min)
- Check trends during peak activity hours
- Share interesting discoveries
- Report bugs with details

❌ **DON'T**:
- Spam refresh button (rate limits)
- Modify code without understanding
- Share misleading trends
- Ignore CORS warnings (expected behavior)

## Uninstallation

### Remove Play App

1. Navigate to your Play list
2. Find "Federated Trends Discovery"
3. Click delete/remove button
4. Confirm deletion

### Clear Cached Data

AiScript data persists in browser localStorage:

1. Open browser DevTools (F12)
2. Go to Application/Storage tab
3. Find localStorage for your instance
4. Look for `trends_snapshots` key
5. Delete the key

Or clear all Play data:
```javascript
// Run in browser console on Play page
localStorage.removeItem('trends_snapshots');
```

## Updating

### To New Version

1. **Backup Current**:
   - Copy your existing code
   - Save configuration changes

2. **Get New Version**:
   - Download latest release
   - Or copy updated code from repository

3. **Replace Code**:
   - Open Play editor
   - Select all existing code
   - Paste new code

4. **Restore Config**:
   - Re-apply your custom `CONFIG` values
   - Save

5. **Test**:
   - Run the updated Play
   - Verify functionality

**Note**: Cached snapshots are compatible across minor versions (1.0 → 1.x)

## Support

### Getting Help

**Check Documentation**:
1. [README.md](./README.md) - Full documentation
2. [SPEC.md](./SPEC.md) - Technical specification
3. [CHANGELOG.md](./CHANGELOG.md) - Version history

**Common Issues**:
- No trends: Wait for activity or try refresh
- CORS errors: Expected for mutual probes, ignore
- Slow loading: Reduce MAX_TAGS_DISPLAY
- Missing media: Posts may not have attachments

**Report Bugs**:
1. Check if issue exists
2. Gather information:
   - Instance version
   - Browser & OS
   - Error messages
   - Steps to reproduce
3. Create detailed issue report

### Community

- **Discussions**: GitHub Issues
- **Updates**: Watch repository for releases
- **Contribute**: PRs welcome!

## Next Steps

After successful installation:

1. ✅ Browse trending hashtags
2. ✅ Discover interesting media
3. ✅ Engage with trending content
4. ✅ Share the app with others
5. ✅ Provide feedback for improvements

---

**Version**: 1.0.0
**Updated**: 2025-11-05
**Estimated Install Time**: 5 minutes
**Difficulty**: Beginner-friendly
