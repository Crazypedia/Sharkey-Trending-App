# Testing Checklist - Federated Trends Discovery

This document provides a comprehensive testing checklist for validating the Federated Trends Discovery Play app.

## Pre-Testing Setup

### Environment Verification

- [ ] Misskey/Sharkey instance version: v12.119+ or v2023.11+
- [ ] Browser: Chrome/Firefox/Safari (latest version)
- [ ] Account permissions: Can create Play apps
- [ ] API endpoints available:
  - [ ] `hashtags/trend`
  - [ ] `notes/search-by-tag` or `notes/search`
  - [ ] `users/following` or `i/following`

### Installation Verification

- [ ] AiScript code copied completely (559 lines)
- [ ] No syntax errors in Play editor
- [ ] Play saved successfully
- [ ] Metadata set (title, description, visibility)

## Functional Testing

### 1. Initial Load

**Test**: First launch of the app

- [ ] App loads without errors
- [ ] Loading state appears briefly
- [ ] Initial data fetches within 5 seconds
- [ ] No console errors (except expected CORS warnings)

**Expected Behavior**:
- ✅ Shows "⏳ Loading trends..." during fetch
- ✅ Transitions to main interface
- ✅ Displays hashtag cloud or "No trending tags" message
- ✅ Shows status bar with counts

### 2. Hashtag Cloud Display

**Test**: Trending hashtags rendering

- [ ] Hashtags appear as clickable buttons
- [ ] Font sizes vary based on popularity
- [ ] Larger font = more popular tag
- [ ] All hashtags start with `#`
- [ ] Text is readable (not too small/large)
- [ ] Tags fit on screen without overflow

**Validation**:
- Min font size ≥ 14px
- Max font size ≤ 36px
- Tag count ≤ 30
- No overlapping text

### 3. Hashtag Interaction

**Test**: Clicking hashtags

- [ ] Click on a hashtag
- [ ] Dialog or modal appears
- [ ] Shows sample posts OR post creation form
- [ ] Post text is readable
- [ ] Dialog can be closed

**Expected Posts**:
- Up to 3 sample posts
- Post text truncated to 100 characters
- Or "(no text)" for media-only posts

### 4. Media Wall Display

**Test**: Trending media rendering

- [ ] "🎬 Trending Media" section appears
- [ ] Media items displayed as buttons
- [ ] Shows 🖼️ for images, 🎥 for videos
- [ ] Up to 25 media items
- [ ] OR "No trending media available" message

**Validation**:
- Media count ≤ 25
- Video embeds ≤ 3
- All items clickable

### 5. Media Interaction

**Test**: Clicking media items

- [ ] Click on a media item
- [ ] Dialog shows post preview
- [ ] Author username visible
- [ ] Post text displayed (if available)
- [ ] Media file count shown
- [ ] Dialog can be closed

### 6. Refresh Functionality

**Test**: Manual refresh

- [ ] Click "🔄 Refresh Trends" button
- [ ] Loading state appears
- [ ] New data fetched (status bar counts update)
- [ ] Hashtag cloud refreshes
- [ ] Media wall refreshes
- [ ] Completes within 5 seconds

**Validation**:
- Tag counts may change
- Media items may change
- No duplicate tags
- No errors

### 7. Status Bar

**Test**: Information display

- [ ] Status bar visible
- [ ] Shows: `Sources: X | Tags: Y | Media: Z`
- [ ] Counts are accurate
- [ ] Updates after refresh

**Expected Values**:
- Sources: Usually 1 (local)
- Tags: 0-30
- Media: 0-25

### 8. Data Persistence

**Test**: Snapshot caching

- [ ] Refresh trends multiple times
- [ ] Close/reload browser tab
- [ ] Reopen Play app
- [ ] Previous trends should still be available
- [ ] Status bar shows data

**Validation**:
- Check browser localStorage (DevTools)
- Look for `trends_snapshots` key
- Should contain up to 12 snapshots
- Each snapshot has timestamp

### 9. Error Handling

**Test**: API failures

**Scenario A**: No trending data
- [ ] Instance with no activity
- [ ] App shows "No trending tags available"
- [ ] Suggests trying refresh

**Scenario B**: Network error
- [ ] Disable network (airplane mode)
- [ ] Click refresh
- [ ] Error handled gracefully
- [ ] No app crash

**Scenario C**: API endpoint missing
- [ ] Older Misskey version
- [ ] Shows appropriate error message
- [ ] Suggests checking instance version

### 10. Mutual Server Detection

**Test**: Federation discovery

- [ ] Check browser console
- [ ] Look for mutual server detection logs
- [ ] Should detect followed users' servers
- [ ] CORS errors expected (not a bug)

**Expected Behavior**:
- Detects up to 8 mutual servers
- Attempts trend endpoint probes
- Most will fail due to CORS (normal)
- Does not block main functionality

## Performance Testing

### 11. Rendering Performance

**Test**: UI responsiveness

- [ ] Hashtag cloud renders smoothly
- [ ] No layout jank or flicker
- [ ] Scrolling is smooth
- [ ] Button clicks respond immediately

**Benchmarks**:
- Initial render: < 3 seconds
- Refresh cycle: < 5 seconds
- Click response: < 500ms
- No memory leaks over 30 minutes

### 12. Mobile Responsiveness

**Test**: Mobile viewport

- [ ] Open app on mobile device or narrow browser
- [ ] Hashtag cloud adapts to width
- [ ] Tags don't overflow screen
- [ ] Buttons are tappable (not too small)
- [ ] Media grid adjusts columns

**Validation**:
- Touch targets ≥ 44px
- Text readable without zoom
- No horizontal scroll
- Controls accessible

### 13. Resource Usage

**Test**: Browser resource consumption

- [ ] Open DevTools > Performance
- [ ] Record session with multiple refreshes
- [ ] Check CPU, memory, network usage

**Acceptable Ranges**:
- Memory: < 50MB for app
- CPU: Spikes during fetch/render, then idle
- Network: Minimal (only API calls)
- No constant polling

## Edge Case Testing

### 14. Empty States

**Test A**: No hashtags trending
- [ ] Instance with zero hashtag activity
- [ ] Shows appropriate message
- [ ] Refresh button still works

**Test B**: No media posts
- [ ] Trending posts without attachments
- [ ] Shows "No trending media available"
- [ ] Doesn't break layout

### 15. Large Datasets

**Test**: Many trending tags

- [ ] Instance with 100+ trending tags
- [ ] App limits to 30 displayed
- [ ] No performance degradation
- [ ] Most popular tags shown

### 16. Special Characters

**Test**: Unusual hashtags

- [ ] Tags with Unicode (emoji, non-Latin)
- [ ] Very long tag names
- [ ] Tags with numbers/underscores

**Expected**:
- All render correctly
- No text overflow
- Clickable and functional

### 17. Rapid Interactions

**Test**: Stress testing

- [ ] Click multiple hashtags quickly
- [ ] Click refresh repeatedly
- [ ] Click multiple media items
- [ ] No crashes or freezes

### 18. Configuration Changes

**Test**: Custom config values

- [ ] Edit `CONFIG` object
- [ ] Change `MAX_TAGS_DISPLAY` to 15
- [ ] Change `FONT_MAX` to 28
- [ ] Save and refresh
- [ ] Verify changes applied

## Browser Compatibility

### 19. Cross-Browser Testing

**Chrome/Edge**:
- [ ] Full functionality
- [ ] No console errors
- [ ] Rendering correct

**Firefox**:
- [ ] Full functionality
- [ ] No console errors
- [ ] Rendering correct

**Safari**:
- [ ] Full functionality
- [ ] No console errors
- [ ] Rendering correct

**Mobile Browsers**:
- [ ] iOS Safari
- [ ] Chrome Mobile
- [ ] Samsung Internet

## Security Testing

### 20. Input Validation

**Test**: API response handling

- [ ] Malformed API responses handled
- [ ] Null/undefined checks present
- [ ] No XSS vulnerabilities in rendered text
- [ ] Safe object property access

### 21. Data Storage

**Test**: localStorage usage

- [ ] Data stored safely
- [ ] No sensitive information leaked
- [ ] Snapshots properly structured
- [ ] No injection attacks possible

## Accessibility Testing

### 22. Keyboard Navigation

**Test**: Keyboard-only usage

- [ ] Tab through interface
- [ ] All buttons focusable
- [ ] Enter/Space activate buttons
- [ ] Focus indicators visible

### 23. Screen Reader

**Test**: Assistive technology

- [ ] Hashtag buttons have labels
- [ ] Status information announced
- [ ] Dialog content accessible
- [ ] Meaningful element roles

## Regression Testing

### 24. After Updates

**Checklist for new versions**:

- [ ] All functional tests pass
- [ ] No new console errors
- [ ] Cached data compatible
- [ ] Configuration preserved
- [ ] Performance unchanged
- [ ] Documentation updated

## Sign-Off

### Testing Summary

**Tested By**: ________________
**Date**: ________________
**Version**: 1.0.0
**Instance**: ________________
**Browser**: ________________

### Results

- [ ] All critical tests passed
- [ ] Minor issues documented
- [ ] Ready for production use

### Known Issues

Document any issues found:

1. _______________________________________________
2. _______________________________________________
3. _______________________________________________

### Notes

_______________________________________________
_______________________________________________
_______________________________________________

---

## Automated Testing (Future)

### Unit Tests (Planned)

```javascript
// Future test suite structure
describe('FederatedTrends', () => {
  test('normalizeScores() returns 0-1 range', () => {
    // Test implementation
  });

  test('calculateFontSize() respects min/max bounds', () => {
    // Test implementation
  });

  test('aggregateTags() combines sources correctly', () => {
    // Test implementation
  });
});
```

### Integration Tests (Planned)

- Mock API responses
- Test full render cycle
- Validate snapshot storage
- Test error scenarios

---

**Testing Guide Version**: 1.0.0
**Last Updated**: 2025-11-05
**Next Review**: After v1.1 release
