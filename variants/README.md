# GTM Traffic Source Variants

This directory contains alternative implementations of the GTM Traffic Source solution for specific use cases.

## Available Variants

### 1. CSP-Compliant Version
**File:** `gtm-traffic-source-csp.html`

A Content Security Policy (CSP) compliant version that loads the script from an external file instead of inline execution.

**Use when:**
- Your website has strict CSP headers
- You need to avoid `'unsafe-inline'` in your CSP policy
- You want to serve the script from a CDN

**Setup:**
1. Upload `gtm-traffic-source-external.js` to your web server
2. Use `gtm-traffic-source-csp.html` in your GTM Custom HTML Tag
3. Update your CSP header to allow the script source

### 2. External Script Version
**File:** `gtm-traffic-source-external.js`

The core JavaScript code without GTM-specific wrapper, suitable for external loading.

**Use when:**
- You want to load the script from a CDN
- You need CSP compliance
- You want to cache the script separately

**Features:**
- Same functionality as the main GTM version
- No GTM-specific wrapper code
- Can be loaded via `<script src="">` tag
- Requires manual MSID integration

## Integration Examples

### CSP-Compliant Setup
```html
<!-- In GTM Custom HTML Tag -->
<script src="https://yourdomain.com/js/gtm-traffic-source-external.js"></script>
```

### External Script Loading
```html
<!-- Direct script loading -->
<script src="https://yourdomain.com/js/gtm-traffic-source-external.js"></script>
<script>
  // Initialize after MSID is ready
  window.addEventListener('msid:ready', function() {
    // Traffic source will be automatically classified
  });
</script>
```

## CSP Headers

For CSP compliance, add these headers to your server:

```
Content-Security-Policy: script-src 'self' https://yourdomain.com; object-src 'none'; base-uri 'self';
```

Replace `https://yourdomain.com` with your actual domain where the script is hosted.

## File Structure

```
variants/
├── README.md                           # This file
├── gtm-traffic-source-csp.html        # CSP-compliant GTM tag
└── gtm-traffic-source-external.js     # External script version
```

## Notes

- All variants provide identical functionality
- Choose the variant that best fits your technical requirements
- The main `tags/gtm-traffic-source.html` remains the recommended version for most use cases
