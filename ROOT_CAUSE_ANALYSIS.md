# Root Cause Analysis: Static Site Export Issues

**Date:** January 27, 2025  
**Site:** All Type Towing (alltypetowing.com)  
**Export Tool:** Simply Static Pro  
**Issue:** Static site missing main content sections and styling

---

## Executive Summary

The static HTML export of the WordPress site is missing critical content sections (hero, phone number, service highlights, "Who We Are", "Quality You Can Trust") and has path-related issues preventing CSS and JavaScript from loading correctly. The export appears incomplete, with content sections 1-8 missing entirely.

---

## Problem Statement

### Primary Issue
The static site (`index.html`) does not display correctly:
- **Missing Content:** Main content sections (hero section, phone number, service highlights, "Who We Are", "Quality You Can Trust") are completely absent
- **Missing Styling:** CSS files fail to load when opened via `file://` protocol
- **Incomplete Export:** HTML structure jumps from header (section 0) directly to location section (section 9), skipping sections 1-8

### Expected vs. Actual

**Expected Structure:**
```
Header (section 0_tb_header)
├── Main Content Area
│   ├── Hero Section (section 1) - Tow truck background, phone number
│   ├── Service Highlights (section 2-3) - "FAST RESPONSE TIME", service badges
│   ├── "Who We Are" Section (section 4-5)
│   ├── "Quality You Can Trust" Section (section 6-7)
│   └── Additional content sections (section 8)
├── Location Section (section 9)
└── Footer (section 0_tb_footer)
```

**Actual Structure:**
```
Header (section 0_tb_header)
├── [MISSING: Sections 1-8]
└── Location Section (section 9)
└── Footer (section 0_tb_footer)
```

---

## Root Causes

### 1. Incomplete Simply Static Export (PRIMARY)

### 1b. Security/Caching Plugins Blocking Crawler (RELATED - CRITICAL)

**Root Cause:** Security and caching plugins (especially Defender Pro) block Simply Static's crawler, causing incomplete exports.

**Evidence:**
- Export completes but content sections are missing
- HTML structure incomplete (sections 1-8 missing)
- Different content than expected in export
- Export appears successful but site looks broken

**Why This Happens:**
1. **Crawler Blocking:** Security plugins block loopback / "bot-like" requests from Simply Static's crawler
2. **Different Markup:** Plugins serve different markup to non-logged-in users or certain user agents
3. **Runtime Dependencies:** Plugins inject optimizations that rely on runtime JavaScript
4. **Cache Mutations:** Cached content interferes with crawler or serves stale/different content

**Common Offenders:**
- **Defender Pro** - Very common cause of crawler blocking
- **WP Smush** - Can interfere with crawler
- **Hustle** - Optimization plugin that may block crawler
- **Wordfence, iThemes Security** - Security plugins that block "bot-like" requests
- **Caching plugins** - May serve cached/different content to crawler

**Impact:** CRITICAL - Export appears successful but content is missing, making it seem like Simply Static "missed" content when actually the crawler was blocked

**Resolution:**
- Disable Defender Pro (or whitelist crawler / allow loopback requests)
- Clear Divi cache before export
- Disable all security/caching plugins before export
- Run fresh full export after disabling plugins
- Test via local web server to verify content

**Fast Isolation Test:**
1. Temporarily disable Defender Pro
2. Clear Divi cache
3. Run fresh full export
4. Preview via `http://localhost:8080/`
5. If content appears, confirms crawler-blocking issue

### 1a. Missing Divi Assets (RELATED - CRITICAL)

**Root Cause:** Divi theme assets, specifically `wp-content/et-cache/` directory, may be missing from export.

**Evidence:**
- Divi generates CSS files in `wp-content/et-cache/` that are referenced in HTML
- If `et-cache/` directory is missing, all CSS files fail to load
- Simply Static has a Divi integration that must be enabled

**Why This Happens:**
1. **Divi Integration Not Enabled:** Simply Static Divi integration may be disabled
2. **Export Settings:** Asset inclusion settings may exclude Divi cache files
3. **Export Errors:** Export may have failed silently for certain directories
4. **Incomplete Export:** Export may not have captured all required directories

**Impact:** CRITICAL - Site will be completely unstyled if Divi CSS files are missing

**Required Directories:**
- `wp-content/et-cache/` - **CRITICAL** - Divi generated CSS
- `wp-content/themes/Divi/` - Divi theme files
- `wp-content/uploads/` - Images and media
- `wp-includes/` - WordPress core assets

**Resolution:**
- Verify Simply Static Divi integration is enabled
- Check export folder/repository for required directories
- Re-export if assets are missing
- Add asset verification to post-export checklist

### 1. Incomplete Simply Static Export (PRIMARY)

**Root Cause:** Simply Static Pro did not export the main page content sections.

**Evidence:**
- HTML file jumps from line 145 (end of header) directly to line 148 (location section)
- No `<div id="main-content">` or `<div id="et-main-area">` containers present
- Section numbering skips from `et_pb_section_0_tb_header` to `et_pb_section_9`
- Content sections 1-8 are completely absent from the HTML

**Why This Happened:**
1. **Page Builder Content Not Detected:** Divi Theme Builder content may not have been recognized by Simply Static
2. **Export Settings:** Homepage may not have been included in export scope
3. **Dynamic Content:** Content may be dynamically generated and not captured in static export
4. **Export Timing:** Export may have occurred before page was fully saved/published
5. **Theme Builder Scope:** Divi Theme Builder sections may have incorrect scope settings

**Impact:** CRITICAL - Site is non-functional without main content

---

### 2. Absolute Path References (SECONDARY - RESOLVED)

**Root Cause:** All CSS, JavaScript, and asset paths used absolute paths (`/wp-content/...`) instead of relative paths.

**Evidence:**
- CSS links: `href="/wp-content/et-cache/2/et-divi-dynamic-tb-133-tb-162-2.css"`
- JavaScript: `src="/wp-includes/js/jquery/jquery.min.js"`
- Image references in CSS: `url(/wp-content/uploads/...)`
- Font references: `url(/wp-content/themes/Divi/...)`

**Why This Happened:**
- Simply Static exports paths as they appear in WordPress (absolute from root)
- No path conversion configured in Simply Static settings
- WordPress uses absolute paths by default for performance

**Impact:** HIGH - Prevents CSS/JS from loading when opened via `file://` protocol or when deployed to subdirectories

**Resolution Status:** ✅ FIXED
- Converted all HTML paths from `/wp-content/` to `wp-content/`
- Converted all HTML paths from `/wp-includes/` to `wp-includes/`
- Fixed inline CSS `url()` functions in HTML
- Fixed CSS files in `wp-content/et-cache/2/` to use relative paths (`../../`)

---

### 3. CSS Preload Mechanism (SECONDARY - RESOLVED)

**Root Cause:** CSS files were loaded using `preload` with JavaScript `onload` handlers instead of standard `stylesheet` links.

**Evidence:**
```html
<link rel="preload" href="..." as="style" onload="this.onload=null;this.rel='stylesheet'">
```

**Why This Happened:**
- Divi theme uses performance optimization technique to defer CSS loading
- Requires JavaScript to execute to convert preload to stylesheet
- May fail in static environments if JavaScript doesn't execute

**Impact:** MEDIUM - CSS may not load if JavaScript fails or is disabled

**Resolution Status:** ✅ FIXED
- Converted all `preload` CSS links to standard `stylesheet` links
- Removed dependency on JavaScript for CSS loading

---

### 4. Plugin-Generated Code (TERTIARY - RESOLVED)

**Root Cause:** Minified, plugin-generated code (Google Reviews widget, WP Smush, Analytics) was included in export.

**Evidence:**
- Google Reviews widget: 200+ lines of minified HTML/JS on single line
- WP Smush lazy-loading scripts
- Matomo/Beehive Analytics tracking code
- Admin-ajax.php references

**Why This Happened:**
- Plugins were active during export
- Simply Static includes all rendered HTML
- No plugin filtering configured

**Impact:** LOW - Increases file size, adds unnecessary code, potential security concerns

**Resolution Status:** ✅ FIXED
- Removed Google Reviews widget HTML
- Removed WP Smush JavaScript files
- Removed Analytics tracking code
- Removed plugin CSS/JS references

---

## Symptoms Observed

### Visual Symptoms
1. **Unstyled Content:** White background, no colors, no layout
2. **Missing Hero Section:** No tow truck background image
3. **Missing Phone Number:** "(443) 740-0423" not visible in hero
4. **Missing Service Highlights:** "FAST RESPONSE TIME" banner missing
5. **Missing Content Sections:** "Who We Are" and "Quality You Can Trust" sections absent
6. **Only Footer Visible:** Footer displays correctly (suggesting some CSS loads)

### Technical Symptoms
1. **Browser Console Errors:**
   ```
   GET file:///C:/wp-content/... net::ERR_FILE_NOT_FOUND
   ```
2. **Missing CSS Files:** CSS files fail to load via `file://` protocol
3. **Missing JavaScript:** jQuery and theme scripts fail to load
4. **Incomplete HTML Structure:** Missing `<div id="main-content">` container

---

## Impact Assessment

| Issue | Severity | Status | Impact |
|-------|----------|--------|--------|
| Missing Main Content Sections | **CRITICAL** | ❌ UNRESOLVED | Site is non-functional, missing all primary content |
| Absolute Path References | HIGH | ✅ RESOLVED | CSS/JS now load correctly with relative paths |
| CSS Preload Mechanism | MEDIUM | ✅ RESOLVED | CSS now loads without JavaScript dependency |
| Plugin-Generated Code | LOW | ✅ RESOLVED | Cleaner code, reduced file size |

---

## Resolution Steps

### Completed Fixes ✅

1. **Path Conversion:**
   - ✅ Converted all absolute paths to relative paths in HTML
   - ✅ Fixed CSS file paths in `wp-content/et-cache/2/`
   - ✅ Fixed inline CSS `url()` functions
   - ✅ Fixed JavaScript file references

2. **CSS Loading:**
   - ✅ Converted `preload` links to standard `stylesheet` links
   - ✅ Removed JavaScript dependency for CSS loading

3. **Code Cleanup:**
   - ✅ Removed Google Reviews widget code
   - ✅ Removed WP Smush scripts
   - ✅ Removed Analytics tracking code

### Required Actions ❌

1. **Re-export from WordPress:**
   - [ ] Verify homepage is published and saved
   - [ ] Check Simply Static export settings
   - [ ] Ensure homepage URL is included in export
   - [ ] Verify Divi Theme Builder scope settings
   - [ ] Perform fresh export with all content sections

2. **Verify Export Settings:**
   - [ ] Check "Additional URLs" includes homepage
   - [ ] Verify "Include all pages" is enabled
   - [ ] Confirm export includes page builder content
   - [ ] Test export on staging before production

3. **Post-Export Validation:**
   - [ ] Verify all content sections are present
   - [ ] Check CSS files load correctly
   - [ ] Test JavaScript functionality
   - [ ] Validate images and assets load
   - [ ] Test on localhost HTTP server
   - [ ] Test on GitHub Pages deployment

---

## Recommendations

### Immediate Actions

1. **Re-export the site** from WordPress using Simply Static
   - Ensure homepage is fully published
   - Include homepage in export settings
   - Verify all Divi sections are included

2. **Verify export completeness** before deploying
   - Check HTML structure includes all sections
   - Validate CSS and JS files are present
   - Test locally via HTTP server (not `file://`)

### Long-term Improvements

1. **Export Process Documentation:**
   - Document Simply Static settings used
   - Create checklist for pre-export validation
   - Establish post-export verification steps

2. **Simply Static URL Configuration:**
   - For production sites: Configure Simply Static to use **Absolute URLs**
   - Set **Destination URL Type** = "Absolute URL"
   - Set **Static URL** = production domain (e.g., `https://alltypetowing.com`)
   - Enable **Force URL replacements**
   - This prevents path issues in production deployment
   - See [Migration Guide - Simply Static Configuration](#) for details

3. **Content Validation:**
   - Create automated checks for missing sections
   - Validate HTML structure completeness
   - Compare export to source WordPress site

4. **Testing Protocol:**
   - Always test exports on localhost HTTP server
   - Verify all content sections are present
   - Test CSS and JavaScript loading
   - Validate responsive design

---

## Technical Details

### File Structure Analysis

**Current `index.html` Structure:**
```html
<div id="page-container">
  <div id="et-boc">
    <header>
      <!-- Section 0: Header (PRESENT) -->
    </header>
    <!-- MISSING: <div id="et-main-area"> -->
    <!-- MISSING: <div id="main-content"> -->
    <!-- MISSING: Sections 1-8 -->
    <div id="map">
      <!-- Section 9: Location (PRESENT) -->
    </div>
    <footer>
      <!-- Section 0: Footer (PRESENT) -->
    </footer>
  </div>
</div>
```

**Expected Structure:**
```html
<div id="page-container">
  <div id="et-boc">
    <header>
      <!-- Section 0: Header -->
    </header>
    <div id="et-main-area">
      <div id="main-content">
        <div class="container">
          <div id="content-area">
            <!-- Section 1: Hero -->
            <!-- Section 2-3: Service Highlights -->
            <!-- Section 4-5: Who We Are -->
            <!-- Section 6-7: Quality You Can Trust -->
            <!-- Section 8: Additional Content -->
          </div>
        </div>
      </div>
    </div>
    <div id="map">
      <!-- Section 9: Location -->
    </div>
    <footer>
      <!-- Section 0: Footer -->
    </footer>
  </div>
</div>
```

### Path Conversion Details

**Before:**
- HTML: `href="/wp-content/et-cache/2/file.css"`
- CSS: `url(/wp-content/uploads/image.jpg)`
- JS: `src="/wp-includes/js/jquery.min.js"`

**After:**
- HTML: `href="wp-content/et-cache/2/file.css"`
- CSS: `url(../../uploads/image.jpg)` (from `et-cache/2/` directory)
- JS: `src="wp-includes/js/jquery.min.js"`

---

## Conclusion

The primary issue is an **incomplete Simply Static export** that failed to capture the main content sections of the homepage. Secondary issues related to path references and CSS loading have been resolved, but the site remains non-functional without the missing content sections.

**Next Steps:**
1. Re-export the site from WordPress
2. Verify all content sections are included
3. Apply path fixes if needed
4. Test thoroughly before deployment

---

## Future Considerations

### GitHub Pages Deployment Configuration

**Potential Issue:**
Current configuration uses:
- Repository: **Private**
- Branch: `main`
- Folder: (blank - root)

**Problem:**
- GitHub Pages from private repos requires **paid GitHub plan** (Pro, Team, or Enterprise)
- GitHub Pages expects either:
  - `gh-pages` branch, OR
  - `/docs` folder on `main` branch
- Root of `main` branch only works for **public repositories**

**Impact:** Site may not deploy to GitHub Pages with current configuration

**Resolution Options:**
1. Switch to `gh-pages` branch in Simply Static settings
2. Use `/docs` folder on `main` branch
3. Make repository public (if acceptable)
4. Upgrade to paid GitHub plan (if private repo required)

**See Also:** [Migration Guide - GitHub Pages Deployment Requirements](#)

## Revision History

- **2025-01-27:** Initial RCA document created
- Documented missing content sections issue
- Documented path conversion fixes
- Documented CSS preload fixes
- Documented plugin code removal
- Added GitHub Pages deployment configuration considerations

---

## Related Documents

- `MIGRATION_GUIDE.md` - Complete migration process documentation
- `SITE_OVERVIEW.md` - Site structure and organization
- `README.md` - Project overview

