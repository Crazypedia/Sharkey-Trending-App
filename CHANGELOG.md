# Changelog

All notable changes to the Federated Trends Discovery Play app will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-11-05

### Added

#### Core Features
- **Trending Hashtag Cloud**: Dynamic display of trending hashtags with font-size scaling (14-36px) based on popularity
- **Interactive Hashtags**: Click-to-view sample posts for any trending hashtag
- **Font Weight Scaling**: Hashtag font weight varies from 400-700 based on popularity
- **Trending Media Wall**: Display of trending images and videos from popular posts
- **Media Click Handlers**: View post details when clicking media items

#### Data Management
- **Snapshot Caching**: Persistent storage using `Mk:save` with up to 12 historical snapshots
- **Automatic Snapshot Cleanup**: Trim old snapshots to maintain performance
- **Data Persistence**: Trends survive browser restarts and page reloads

#### API Integration
- **Local Trends Fetching**: Uses `Mk:api('hashtags/trend')` to get local trending tags
- **Post Retrieval**: Fetches sample posts via `notes/search-by-tag` with fallback to `notes/search`
- **Mutual Server Detection**: Identifies federated servers from following list (foundation for future features)
- **Multiple API Fallbacks**: Graceful degradation when primary endpoints unavailable

#### UI Components
- **Manual Refresh Button**: User-triggered trend updates
- **Status Bar**: Shows active sources, tag count, and media count
- **Loading States**: Visual feedback during data fetching
- **Error States**: Clear error messages with recovery suggestions

#### Configuration
- **Customizable Limits**:
  - Max snapshots: 12
  - Max tags displayed: 30
  - Max media items: 25
  - Font size range: 14-36px
- **Performance Tuning**: Configurable limits for optimal rendering

### Technical Implementation

#### Data Structures
- Snapshot storage with timestamp tracking
- Tag normalization and scoring algorithm
- Media post collection and filtering
- Source weight system (prepared for federation)

#### Algorithms
- Tag score normalization (0-1 range)
- Bubble sort for tag ranking (AiScript-compatible)
- Font size calculation with min/max bounds
- Font weight calculation for emphasis

#### Error Handling
- Try-catch blocks for all API calls
- Null checks for object access
- Fallback endpoints for critical operations
- Graceful degradation when features unavailable

### Known Limitations

- **No Automatic Refresh**: AiScript Play limitations prevent `setInterval` usage
- **Local Only (v1.0)**: Mutual server trend aggregation prepared but not activated
- **CORS Restrictions**: External server probing may fail due to browser security
- **Limited Video Embeds**: AiScript UI constraints limit embed functionality
- **Basic Sorting**: Uses bubble sort due to AiScript standard library limitations

### Performance

- Renders up to 30 hashtags efficiently
- Handles 25 media items without lag
- Snapshot storage capped at reasonable size
- No memory leaks from event handlers

### Browser Compatibility

- Tested on modern browsers (Chrome, Firefox, Safari)
- Mobile-responsive layout
- Works in Misskey/Sharkey embedded environment

### Security

- No external script loading
- No eval or code injection vectors
- Safe object property access
- Input validation on API responses

## [Unreleased]

### Planned for v1.1

- Enhanced error messaging with recovery actions
- Custom CSS theme support
- Improved mobile layout optimization
- Hashtag exclude/filter list
- Better video preview handling

### Planned for v2.0

- Active mutual server trend aggregation
- Weighted source blending UI
- Time window selection dropdown
- View mode toggle (cloud/list/grid)
- Export trends data functionality
- Hashtag history visualization

### Planned for v3.0

- Advanced filtering and sorting options
- Hashtag trend charts over time
- Multi-instance comparison view
- Custom refresh interval configuration
- Bookmarked hashtag tracking
- Notification system for trending topics

## Development Notes

### Version 1.0.0 Development Timeline

**Week 1**: Core Architecture
- Designed data structures and state management
- Implemented storage layer with `Mk:save/load`
- Created API abstraction layer

**Week 2**: Hashtag Cloud
- Built tag aggregation and normalization
- Implemented font-size scaling algorithm
- Added click handlers and post preview

**Week 3**: Media Wall
- Created media post collection logic
- Implemented responsive grid layout
- Added media click handlers

**Week 4**: Polish & Testing
- Error handling and fallbacks
- Documentation (README, CHANGELOG)
- Testing on live instance
- Performance optimization

### Technical Decisions

**Why AiScript-Only?**
- Meets SPEC-001 requirement for no external servers
- Runs entirely in Misskey Play environment
- Maximum privacy and control

**Why Manual Refresh?**
- AiScript Play does not support `setInterval`
- Ensures user control over API usage
- Prevents rate limiting issues

**Why Local-Only (v1.0)?**
- CORS policies block most external probes
- Ensures reliable functionality
- Foundation prepared for future proxy solutions

**Why Bubble Sort?**
- AiScript lacks native sort function
- Simple implementation, sufficient for <30 items
- O(n²) acceptable for small datasets

### Migration Notes

**From Earlier Versions**: N/A (initial release)

**To v1.1**: Will be backward compatible
- Existing snapshots will continue to work
- Configuration may expand with defaults

**To v2.0**: May require cache clear
- Snapshot format may change for federation
- Will provide migration helper if needed

## Contributing

### How to Report Issues

1. Check existing issues for duplicates
2. Provide Misskey/Sharkey version
3. Include browser and OS details
4. Describe expected vs actual behavior
5. Provide reproduction steps
6. Include console errors if applicable

### How to Contribute Code

1. Fork the repository
2. Create feature branch from main
3. Follow AiScript style guidelines
4. Test thoroughly on live instance
5. Update CHANGELOG.md with changes
6. Submit PR with detailed description

### Code Style

- Use descriptive function names with `@` prefix
- Comment complex algorithms
- Use consistent indentation (2 spaces)
- Keep functions focused and small
- Add error handling for API calls

## Acknowledgments

- **Misskey Project**: For the excellent platform and AiScript language
- **Sharkey Team**: For enhanced features and API improvements
- **Community**: For testing and feedback

---

**Repository**: [Sharkey-Trending-App](https://github.com/Crazypedia/Sharkey-Trending-App)
**License**: Open Source
**Maintainer**: Development Team
