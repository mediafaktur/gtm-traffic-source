# GTM Traffic Source

A lightweight, consent-controlled GTM solution for precise traffic source attribution within marketing sessions. Extends the [Marketing Session ID (MSID)](https://github.com/mediafaktur/gtm-marketing-session-id) solution with comprehensive traffic source classification and storage.

## Overview

For detailed decision logic, classification rules, and business rationale, see this document:<br>
→ 📋 [Traffic Source Strategy](STRATEGY.md)

### Problem
Traditional analytics solutions struggle with accurate traffic source attribution due to:
- **Session persistence issues** - Traffic sources persist across unintended sessions
- **Attribution conflicts** - Click IDs, UTMs, and organic signals create conflicting attributions
- **Cross-tab complexity** - Traffic sources don't carry over properly between tabs
- **Inconsistent classification** - Different tools use different logic for the same traffic
- **UTM parameter errors** - Manual UTM setup leads to incorrect attribution
- **Click ID persistence** - GCLID and other click IDs persist in bookmarks and URLs

### Solution
GTM Traffic Source provides:
- **Session-scoped attribution** - Traffic sources are tied to marketing sessions, not technical sessions
- **Comprehensive classification** - Handles Click IDs, UTMs, organic search, social, referral, and direct traffic
- **Plausibility checks** - Prevents conflicting attributions (e.g., `gclid` + `utm_source=facebook`)
- **Cross-tab carry-over** - Traffic sources persist across tabs within the same marketing session
- **Cookie-free storage** - Uses `sessionStorage` and `localStorage` for reliable data persistence

## What Problems Does TS Solve?

| Problem | Traditional Solutions | TS Solution |
|:---------|:----------------------|:-------------|
| **Session Persistence** | Sources persist in cookies, causing false attribution | Session-scoped storage tied to marketing sessions |
| **Signal Conflicts** | Last-click wins, inconsistent results | Priority-based system with plausibility checks |
| **Cross-Tab Loss** | Each tab starts fresh, losing attribution | Cross-tab carry-over within marketing sessions |
| **UTM Errors** | Manual setup leads to typos and conflicts | Plausibility validation against channel groups |
| **Bookmark Persistence** | GCLID persists in bookmarked URLs | Session-scoped attribution prevents false attribution |
| **Logic Inconsistency** | Different tools, different rules | Standardized BigQuery-compatible logic |
| **Organic/Paid Confusion** | Organic signals override paid signals | Exclusion logic prioritizes paid over organic |
| **Self-Referral Issues** | Internal navigation counted as referral | eTLD+1 comparison excludes self-referrals |

## Features

### 🎯 Attribution Logic
- **Priority-based Classification** - Click IDs → UTMs → Organic → Social → Referral → Direct
- **Plausibility Validation** - Prevents conflicting attributions between sources
- **Cross-tab Carry-over** - Traffic sources persist across tabs within sessions
- **Industry Standards** - Matches Google Analytics standard channel grouping

### 🔧 Technical Features
- **Event-driven Architecture** - Robust MSID integration with race condition prevention
- **Idempotent Processing** - No duplicate classification within sessions
- **Feature Detection** - Polyfills for older browsers (IE11+)
- **UTM Normalization** - Automatic trimming, case handling, and synonym mapping
- **Soft Alias Support** - `utm_medium=paid` automatically normalized to `paid_social` for social platforms
- **Video Platform Detection** - YouTube, Vimeo, etc. excluded from social soft alias
- **Timestamp Consistency** - Uses MSID timestamps for all events

### 🌐 Platform & Compatibility
- **Multi-platform Support** - Google Ads, Facebook, Microsoft, TikTok, LinkedIn, Twitter, etc.
- **Cross-Browser Support** - Chrome 49+, Firefox 45+, Safari 10.1+, Edge (Chromium), IE11 (best-effort)
- **Fallback Strategy** - Graceful degradation for older browsers with polyfills

### 📊 Data & Debugging
- **Debug-friendly** - Comprehensive logging and dataLayer events
- **Conflict Detection** - Flags plausibility conflicts for debugging
- **Meta-Fields** - Version tracking and source-of-truth identification
- **Configurable Logging** - Production-safe debug mode (`CONFIG.DEBUG = false`)
- **Easy Integration** - Single GTM tag, simple form integration

## Advanced Features

### UTM Normalization
Automatic data quality improvements for UTM parameters:
- **Trimming** - Removes leading/trailing whitespace
- **Case Handling** - Converts to lowercase for consistent comparison
- **Synonym Mapping** - Maps common variations to standard values:
  - `cpc`, `ppc` → `paid_search`
  - `paidsocial` → `paid_social`
  - `social_paid` → `paid_social`
  - `paidsearch` → `paid_search`
  - `remarketing` → `display_remarketing`

### Soft Alias Support
Marketing-friendly UTM parameter shortcuts:
- **`utm_medium=paid`** - Automatically normalized to `paid_social` for social platforms
- **Video Platform Detection** - YouTube, Vimeo, etc. are excluded from social soft alias
- **Transparency** - `soft_alias_used: true` flag indicates normalization occurred

### UTM Source Fallback
Intelligent classification when only `utm_source` is provided:
- **Social Sources** (`facebook`, `instagram`, `linkedin`, etc.) → `organic_social` + `social`
- **Video Platforms** (`youtube`, `vimeo`, `twitch`, etc.) → `organic_video` + `video`
- **Search Engines** (`google`, `bing`, `yahoo`, etc.) → `organic_search` + `organic`
- **Debug Logging** - Shows when source-driven fallback is applied

### Enhanced Debugging
Production-ready monitoring and debugging:
- **Version Tracking** - `classifier_version: '1.0.0'` for compatibility checks
- **Source of Truth** - `source_of_truth` indicates classification method used
- **Configurable Logging** - `CONFIG.DEBUG` flag controls console output
- **Conflict Detection** - Flags plausibility conflicts for data quality monitoring
- **UTM Validation** - Comprehensive validation of UTM parameter combinations
- **Unknown Medium Handling** - Invalid UTM combinations set to `medium: 'unknown'`

### Data Consistency
Improved data quality and consistency:
- **E-Mail Classification** - Mail clients (referrer-based) correctly classified as `medium: 'email'` (not `organic`)
- **Channel Group Alignment** - Medium values match their corresponding channel groups
- **Fallback Intelligence** - Smart classification prevents `unassigned` results
- **UTM Normalization** - All medium values are normalized to canonical forms (e.g., `cpc` → `paid_search`)

## Benefits

### Technical Benefits
- **Modular** - TS-Plugin can be developed independently
- **Idempotent** - No duplicate classification per session
- **MSID-compatible** - Uses existing session logic
- **Cross-tab capable** - Carry-over between tabs
- **Plausible** - Prevents conflicting attributions
- **Performance-optimized** - Event-driven with polling fallback

### Integration Benefits
- **GTM Tag runs after MSID Tag** - Proper execution order
- **No dependencies except MSID** - Clean architecture
- **Simple form integration** - Access via `ms_ts_current`
- **Consent-controlled** - Respects user privacy preferences
- **Debug-friendly** - Comprehensive logging and dataLayer events

### Business Benefits
- **Accurate attribution** - Reliable traffic source data
- **Consistent reporting** - Standardized classification across tools
- **Better ROI measurement** - Precise campaign attribution
- **Reduced data quality issues** - Plausibility checks prevent errors
- **Cross-tab continuity** - Complete user journey tracking

## Architecture

### Traffic Source Classification
Priority-based classification system:

1. **Click IDs** (highest priority)
   - `gclid`, `gbraid`, `wbraid`, `msclkid`, `fbclid`, `ttclid`, `li_fat_id`, `twclid`
   - Platform-specific attribution (Google Ads, Facebook, Microsoft, TikTok, LinkedIn, Twitter)

2. **UTM Parameters** (with plausibility checks)
   - `utm_source`, `utm_medium`, `utm_campaign`, `utm_term`, `utm_content`
   - Validated against channel group to prevent conflicts
   - Automatic normalization (trimming, case handling)

3. **`organic_search`** (referrer analysis)
   - Google, Bing, Yahoo, DuckDuckGo, Yandex, Baidu, Ecosia, Qwant, etc.
   - Search term extraction from referrer URLs

4. **Social Media** (paid/organic detection)
   - Facebook, Instagram, LinkedIn, TikTok, Twitter, Pinterest, YouTube, Reddit, etc.
   - Channel group determination based on context

5. **Referral Traffic**
   - External domains (excluding self-referrals)
   - Domain-based source identification

6. **Direct Traffic** (fallback)
   - No referrer, no parameters
   - Default attribution

### Storage Strategy
- **`sessionStorage['ms_ts_current']`** - Tab-scoped truth, easy form integration
- **`sessionStorage['ms_ts:<SID>']`** - Per-session storage for explicit binding
- **`sessionStorage['ms_entryCaptured:<SID>']`** - Idempotency flag per session
- **`localStorage['ms_ts_by_sid:<SID>']`** - Cross-tab carry-over data

### Cross-Tab Carry-over
Traffic sources are carried over to new tabs when:
- **`carryAllowed: true`** - No timeout and no external referrer
- **Same marketing session** - Within 30-minute window
- **No re-classification** - TS is copied from localStorage

### Browser Compatibility
- **IE11+ Support** - Feature detection with polyfills
- **Modern APIs** - URLSearchParams, URL constructor, CustomEvent
- **Fallback Functions** - Custom implementations for older browsers
- **Performance Optimized** - Only loads polyfills when needed

### Event-Driven Communication
- **MSID sends `msid:ready` event** when session is ready
- **TS Plugin listens** for event with polling fallback
- **Race condition prevention** - Robust MSID detection
- **Timestamp consistency** - Uses MSID timestamps for all events

## Quick Start Guide

### 1. Install MSID First
GTM Traffic Source requires the [Marketing Session ID (MSID)](https://github.com/mediafaktur/gtm-marketing-session-id) solution to be installed first.

#### MSID Dependency
GTM Traffic Source depends on MSID for session management:

```javascript
// MSID provides session signals:
window.__ms = {
  sessionId: 'session_1758043189465_198na7mp',
  _signals: {
    isStart: true,           // New session started?
    inactiveTooLong: true,   // 30min timeout exceeded?
    isExternalRef: true,     // External referrer detected?
    carryAllowed: false      // Cross-tab carry-over allowed?
  }
}
```

**Session State Detection:**
- **`isStart: true`** - New marketing session started
- **`inactiveTooLong: true`** - 30-minute timeout exceeded
- **`isExternalRef: true`** - External referrer detected
- **`carryAllowed: false`** - Cross-tab carry-over not allowed

### 2. Add GTM Traffic Source Tag
1. Create a new **Custom HTML Tag** in GTM
2. Copy the content from `tags/gtm-traffic-source.html`
3. Set trigger to **All Pages**
4. Set firing priority to run **after MSID tag**

**Alternative Variants:**
- **CSP-Compliant:** Use `variants/gtm-traffic-source-csp.html` for strict CSP environments
- **External Script:** Use `variants/gtm-traffic-source-external.js` for CDN loading
- See `variants/README.md` for detailed setup instructions

### 3. Access Traffic Source Data

#### **Storage Entries**
The system creates several storage entries for different purposes:

| **Storage Key** | **Storage Type** | **Purpose** | **Example** |
|-----------------|------------------|-------------|-------------|
| `ms_ts_current` | sessionStorage | Current tab's traffic source (easy form access) | `{"source":"google","medium":"cpc",...}` |
| `ms_ts:<SID>` | sessionStorage | Per-session storage (explicit session binding) | `ms_ts:session_1758043189465_198na7mp` |
| `ms_entryCaptured:<SID>` | sessionStorage | Idempotency flag (prevents duplicate classification) | `ms_entryCaptured:session_1758043189465_198na7mp` |
| `ms_ts_by_sid:<SID>` | localStorage | Cross-tab carry-over data | `ms_ts_by_sid:session_1758043189465_198na7mp` |

#### **Traffic Source Data Object**
The complete traffic source object contains all attribution data:

```javascript
// Get current traffic source
var trafficSource = JSON.parse(sessionStorage.getItem('ms_ts_current'));

// Complete data structure
{
  // Basic identification
  "id": "ts_1758043189465_abc123",           // Unique TS ID
  "timestamp": 1758043189465,                // Classification timestamp
  "entry_url": "https://example.com/page",   // URL where user entered
  "referrer": "https://google.com/search",   // Where user came from

  // Classification results
  "method": "click_id",                      // How TS was classified
  "channel_group": "paid_search",            // High-level channel category
  "source": "google",                        // Traffic source platform
  "medium": "cpc",                          // Traffic medium type
  "campaign": "winter2024",                 // Campaign name
  "term": "winter boots",                   // Search term or keyword
  "content": "ad_variant_a",                // Content variant
  "ad_platform": "google_ads",              // Advertising platform
  "click_id": "Cj0KCQiA...",               // Click ID if available

  // Raw parameters (for debugging)
  "raw_params": {
    "utm_source": "google",
    "utm_medium": "cpc",
    "utm_campaign": "winter2024",
    "utm_term": "winter boots",
    "utm_content": "ad_variant_a"
  },

  // Quality and debugging flags
  "classifier_version": "1.0.0",            // Version for compatibility checks
  "source_of_truth": "click_id",            // Classification method used
  "plausibility_conflict": false,           // UTM conflict detected
  "conflict_reason": "",                    // Reason for conflict
  "soft_alias_used": false,                 // UTM normalization applied
  "missing_medium": false                   // Medium was missing/invalid
}
```

#### **Accessing Specific Fields**
```javascript
// Get current traffic source
var trafficSource = JSON.parse(sessionStorage.getItem('ms_ts_current'));

// Core attribution data
console.log(trafficSource.source);          // 'google'
console.log(trafficSource.medium);         // 'cpc' (normalized)
console.log(trafficSource.campaign);       // 'winter2024'
console.log(trafficSource.channel_group);  // 'paid_search'

// Classification metadata
console.log(trafficSource.method);         // 'click_id', 'utm', 'organic', etc.
console.log(trafficSource.source_of_truth); // 'click_id', 'utm', 'referrer', 'direct'

// Quality flags
if (trafficSource.plausibility_conflict) {
  console.warn('UTM conflict detected:', trafficSource.conflict_reason);
}
if (trafficSource.missing_medium) {
  console.warn('Medium was missing or invalid');
}
```

### 4. Practical Use Cases

#### **Form Integration**
```html
<!-- Hidden form fields automatically populated -->
<input type="hidden" name="traffic_source" id="traffic_source">
<input type="hidden" name="traffic_medium" id="traffic_medium">
<input type="hidden" name="traffic_campaign" id="traffic_campaign">

<script>
// Populate form fields
var ts = JSON.parse(sessionStorage.getItem('ms_ts_current'));
document.getElementById('traffic_source').value = ts.source;
document.getElementById('traffic_medium').value = ts.medium;
document.getElementById('traffic_campaign').value = ts.campaign;
</script>
```

#### **Analytics Integration**
```javascript
// Send to Google Analytics 4
gtag('event', 'traffic_source_classified', {
  'traffic_source': trafficSource.source,
  'traffic_medium': trafficSource.medium,
  'traffic_campaign': trafficSource.campaign,
  'channel_group': trafficSource.channel_group,
  'click_id': trafficSource.click_id
});

// Send to custom analytics
analytics.track('Page View', {
  'traffic_source': trafficSource.source,
  'traffic_medium': trafficSource.medium,
  'traffic_campaign': trafficSource.campaign,
  'classification_method': trafficSource.method,
  'source_of_truth': trafficSource.source_of_truth
});
```

#### **CRM Integration**
```javascript
// Send to CRM system
var crmData = {
  'lead_source': trafficSource.source,
  'lead_medium': trafficSource.medium,
  'campaign_name': trafficSource.campaign,
  'ad_platform': trafficSource.ad_platform,
  'search_term': trafficSource.term,
  'content_variant': trafficSource.content,
  'entry_url': trafficSource.entry_url,
  'referrer': trafficSource.referrer
};

// Example: Salesforce
salesforce.createLead(crmData);
```

#### **A/B Testing Integration**
```javascript
// Use traffic source for experiment targeting
var experimentGroup = 'control';
if (trafficSource.channel_group === 'paid_search' && trafficSource.medium === 'cpc') {
  experimentGroup = 'search_ads_test';
} else if (trafficSource.channel_group === 'paid_social') {
  experimentGroup = 'social_ads_test';
}

// Apply experiment
document.body.classList.add('experiment-' + experimentGroup);
```

#### **Cross-Tab Data Sharing**
```javascript
// Check if traffic source exists in other tabs
function getTrafficSourceFromOtherTabs() {
  var sessionId = window.__ms?.sessionId;
  if (!sessionId) return null;

  var carryOverData = localStorage.getItem('ms_ts_by_sid:' + sessionId);
  if (carryOverData) {
    return JSON.parse(carryOverData);
  }
  return null;
}

// Use in new tab
var ts = getTrafficSourceFromOtherTabs();
if (ts) {
  console.log('Traffic source carried over from other tab:', ts);
}
```

#### **Data Quality Monitoring**
```javascript
// Monitor data quality issues
function checkDataQuality(trafficSource) {
  var issues = [];

  if (trafficSource.plausibility_conflict) {
    issues.push('UTM conflict: ' + trafficSource.conflict_reason);
  }

  if (trafficSource.missing_medium) {
    issues.push('Missing or invalid medium');
  }

  if (trafficSource.source === 'unknown') {
    issues.push('Unknown traffic source');
  }

  if (issues.length > 0) {
    console.warn('Data quality issues detected:', issues);
    // Send to monitoring system
    monitoring.report('traffic_source_quality_issues', {
      'issues': issues,
      'traffic_source': trafficSource
    });
  }
}

// Check on page load
var ts = JSON.parse(sessionStorage.getItem('ms_ts_current'));
if (ts) {
  checkDataQuality(ts);
}
```

## Integration Examples

### Basic GTM Setup
```javascript
// GTM Custom HTML Tag
<script>
// Copy content from gtm-traffic-source.html
// This will automatically classify traffic source
// and store it in sessionStorage['ms_ts_current']
</script>
```

### DataLayer Integration
```javascript
// Traffic source data is automatically pushed to dataLayer
window.dataLayer.push({
  'event': 'traffic_source_classified',
  'traffic_source': {
    'source': 'google',
    'medium': 'cpc',
    'campaign': 'winter2024',
    'channel_group': 'paid_search',
    'method': 'click_id'
  }
});
```

### Custom Event Listening
```javascript
// Listen for traffic source classification via dataLayer
window.dataLayer = window.dataLayer || [];
window.dataLayer.push(function() {
  this.addEventListener('traffic_source_classified', function(event) {
    var trafficSource = event.traffic_source;
    console.log('Traffic source classified:', trafficSource);

    // Send to your analytics platform
    analytics.track('Traffic Source Classified', trafficSource);
  });
});
```

## 📚 Documentation

### Strategy & Decision Logic
For comprehensive information about how the system works, see the **[Traffic Source Strategy Document](STRATEGY.md)**:

- 🎯 **Classification Decision Tree** - Complete logic flow for all scenarios
- 📊 **Traffic Source Matrix** - All possible combinations and outcomes
- ⚠️ **Conflict Resolution** - How technical vs. marketing signals are handled
- 🔍 **Plausibility Checks** - Channel group validation rules and examples
- 📋 **Reference Tables** - Complete parameter mappings and use cases
- 🏗️ **Architecture Principles** - Why decisions were made and how they work

### Quick Reference
- **Click IDs** - `gclid`, `gbraid`, `wbraid`, `msclkid`, `fbclid`, `ttclid`, `li_fat_id`, `twclid`
- **Channel Groups** - `paid_search`, `paid_social`, `paid_shopping`, `paid_video`, `display`, `display_remarketing`, `organic_search`, `organic_social`, `organic_video`, `organic_shopping`, `email`, `affiliate`, `referral`, `direct`, `unassigned`
- **Storage Keys** - `ms_ts_current`, `ms_ts:<SID>`, `ms_entryCaptured:<SID>`
- **Events** - `traffic_source_classified` (dataLayer), `msid:ready`

## Browser Compatibility

### Supported Browsers
- **Chrome** 60+ (full support)
- **Firefox** 55+ (full support)
- **Safari** 10.1+ (full support)
- **Edge** 79+ (full support)
- **Internet Explorer** 11+ (with polyfills)

### Polyfills Included
- `URLSearchParams` - For URL parameter parsing
- `URL` constructor - For URL manipulation
- `CustomEvent` - For event dispatching
- `addEventListener`/`removeEventListener` - For event handling
- `String.prototype.includes` - For string operations
- `Array.prototype.includes` - For array operations

### Feature Detection
The script automatically detects browser capabilities and only loads polyfills when necessary, ensuring optimal performance on modern browsers.

## Custom URL Parameters

GTM Traffic Source supports custom URL parameters that are stored both on the root level of the traffic source object and in the `raw_params` for easy access.

### Configuration

Define custom parameters in the `CONFIG.CUSTOM_PARAMS` array:

```javascript
// Example: ['adgroupid', 'location', 'device_type', 'placement']
CUSTOM_PARAMS: []
```

**Important Notes:**
- Custom parameters are automatically prefixed with `cp_` to avoid conflicts with reserved fields
- Parameters are case-insensitive (use lowercase in configuration for clarity)
- Reserved fields (id, source, medium, etc.) are automatically skipped

### Data Structure

Custom parameters are added to both the root level and `raw_params`:

```javascript
{
  // Standard traffic source properties
  "id": "ts_1758043189465_abc123",
  "source": "google",
  "medium": "cpc",
  "campaign": "winter2024",
  "channel_group": "paid_search",

  // Custom parameters on root level (prefixed with cp_)
  "cp_adgroupid": "ag123",
  "cp_location": "de",
  "cp_device_type": "mobile",
  "cp_placement": "search",

  // Raw parameters (includes custom parameters)
  "raw_params": {
    "utm_source": "google",
    "utm_medium": "cpc",
    "utm_campaign": "winter2024",
    "adgroupid": "ag123",
    "location": "de",
    "device_type": "mobile",
    "placement": "search"
  }
}
```

### Usage

```javascript
// Direct access to custom parameters (note the cp_ prefix)
var trafficSource = JSON.parse(sessionStorage.getItem('ms_ts_current'));
var adgroupid = trafficSource.cp_adgroupid;
var location = trafficSource.cp_location;
var deviceType = trafficSource.cp_device_type;

// Form integration
document.getElementById('adgroupid').value = trafficSource.cp_adgroupid || '';
document.getElementById('location').value = trafficSource.cp_location || '';
```

## Performance & Maintenance

### Automatic Cleanup

GTM Traffic Source automatically cleans up old session data from localStorage to maintain optimal performance:

- **When**: Only on new session start (not on page reloads)
- **What**: Removes old `ms_ts_by_sid:*` keys (keeps current session)
- **Limit**: Maximum 10 keys per cleanup (prevents performance impact)
- **Benefit**: Keeps localStorage lean and fast

```javascript
// Cleanup runs automatically on session start
if (signals.isStart) {
  cleanupOldSessionKeys(sessionId);  // Removes old session data
}
```

**Debug Information:**
```javascript
// Console output when cleanup occurs
TS Classifier: Cleaned up 5 old session keys from localStorage
```

## Troubleshooting

### Common Issues

#### 1. Traffic Source Not Classified
- **Check MSID dependency** - Ensure MSID script is loaded first
- **Verify GTM trigger** - Tag should fire on All Pages
- **Check browser console** - Look for error messages
- **Test with debug mode** - Enable detailed logging

#### 2. UTM Parameters Not Working
- **Check parameter format** - Use lowercase, no spaces
- **Verify plausibility** - Source/medium must match channel group
- **Test with Click ID** - UTM may be overridden by Click ID
- **Check conflict flags** - Look for `plausibility_conflict` in data

#### 3. Cross-Tab Carry-over Issues
- **Check session timing** - Must be within 30-minute window
- **Verify localStorage** - Check if data is stored correctly
- **Test session state** - Ensure MSID is working properly
- **Check carryAllowed** - Must be true for carry-over

### Debug Information
The script provides comprehensive debug information in the browser console:
- **Classification method** - How traffic source was determined
- **Conflict detection** - Any plausibility conflicts found
- **Storage operations** - What data is stored where
- **Event dispatching** - When events are fired

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Support

For technical support and questions:
- **GitHub Issues** - Report bugs and request features
- **Documentation** - Check README and strategy documents
- **Code Examples** - See integration examples above

## Author

**/ MEDIAFAKTUR** – Marketing Performance Precision
[https://mediafaktur.marketing](https://mediafaktur.marketing)<br>
Florian Pankarter, [fp@mediafaktur.marketing](mailto:fp@mediafaktur.marketing)

---

**GTM Traffic Source** - Reliable, accurate traffic source classification for Google Tag Manager.
