# WordPress to Static Site Migration Guide

Complete documentation for migrating WordPress sites to static GitHub repositories using Simply Static Pro.

---

## Table of Contents

1. [Pre-Migration Checklist](#pre-migration-checklist)
2. [Plugin Management](#plugin-management)
3. [Standard Operating Procedure](#standard-operating-procedure)
4. [CSS Breaking Changes & Fixes](#css-breaking-changes--fixes)
5. [Troubleshooting Guide](#troubleshooting-guide)
6. [Post-Migration Verification](#post-migration-verification)

---

## Pre-Migration Checklist

Complete all items before starting the migration:

- [ ] **Disable Defender Pro** (or whitelist crawler / allow loopback) - CRITICAL - blocks crawler
- [ ] **Disable Hummingbird plugin** (Simply Static conflicts - Simply Static will warn if active)
- [ ] **Disable problematic plugins** (see [Plugin Management](#plugin-management) section below)
- [ ] **Clear Divi cache** (Divi → Theme Options → Performance → Clear) - CRITICAL
- [ ] **Set Cloudflare to Developer Mode** (disable caching during migration)
- [ ] **Verify Simply Static Pro is installed and activated**
- [ ] **Enable Simply Static Divi integration** (ensures Divi assets are included)
- [ ] **Create GitHub Personal Access Token (PAT)**
- [ ] **Get GitHub org admin approval for PAT** (if using organization account)
- [ ] **Create GitHub repository with README** (Simply Static needs repository to exist first)
- [ ] **Plan GitHub Pages configuration** (private repos need paid plan OR use `gh-pages` branch OR `/docs` folder)
- [ ] **Configure Simply Static URL replacement settings** (General → Replacing URLs: Absolute URL, Static URL = production domain)
- [ ] **Configure Simply Static GitHub deployment settings** (branch: `gh-pages` OR folder: `docs` for private repos)
- [ ] **Test export on staging site first** (before production migration)
- [ ] **Verify WPMUDev Hub is accessible** (if needed for site management)
- [ ] **Test static export via local web server** (use `py -m http.server 8080`, not double-click)

---

## Plugin Management

### Critical: Plugins to Disable Before Export

These plugins cause issues in static HTML exports and should be **disabled** before running Simply Static:

#### 1. WP Smush Pro (Image Optimization)
- **Problem:** Adds lazy-loading JavaScript that references `/wp-admin/admin-ajax.php` (won't work in static HTML)
- **Evidence:** Adds `smush-detector.js` and `smush-lazy-load.min.js` scripts
- **Impact:** Broken lazy-loading functionality, unnecessary JavaScript bloat
- **Action:** **Disable before export** - images are already optimized

#### 2. Beehive Analytics / Matomo (Analytics)
- **Problem:** Adds tracking scripts that won't work properly in static HTML
- **Evidence:** Matomo analytics code with `_paq` tracking variables
- **Impact:** Unnecessary JavaScript, potential broken tracking
- **Action:** **Disable before export** - add Google Analytics directly in static HTML instead

#### 3. SmartCrawl SEO (WPMU Dev)
- **Problem:** Adds SEO meta tags but also unnecessary overhead
- **Evidence:** "SEO meta tags powered by SmartCrawl" comment in HTML
- **Impact:** Minor bloat, but meta tags are already exported
- **Action:** **Disable before export** - meta tags are already in the static HTML

#### 4. Security Plugins (CRITICAL - Crawler Blocking)
- **Examples:** Defender Pro, Wordfence, iThemes Security, WP Defender
- **Problem:** 
  - Block loopback / "bot-like" requests from Simply Static's crawler
  - Serve different markup to non-logged-in users / certain user agents
  - Inject optimizations that rely on runtime JavaScript
  - Can cause incomplete exports or missing content
- **Action:** **Disable before export** (or whitelist crawler / allow loopback requests)
- **Note:** This is a common cause of incomplete exports - security plugins often block the crawler

#### 5. Caching Plugins
- **Examples:** WP Super Cache, W3 Total Cache, Hummingbird
- **Problem:** Not needed - static export is already "cached"
- **Action:** **Disable before export**

#### 6. Backup Plugins
- **Examples:** Snapshot Backups, UpdraftPlus
- **Problem:** Not needed for static export
- **Action:** **Disable before export**

#### 7. Contact Form Plugins
- **Examples:** Forminator, Contact Form 7, Gravity Forms
- **Problem:** Forms won't work in static HTML (no backend processing)
- **Action:** **Disable before export** - replace with static contact links or third-party form service

### Plugins to Keep Enabled

These plugins should remain **active** during export:

#### 1. Widget Google Reviews
- **Status:** Keep enabled if you need reviews in your static site
- **Note:** Generates minified HTML (messy code, but functional)
- **Alternative:** Can be replaced with custom HTML if desired

#### 2. Divi Theme
- **Status:** **Required** - keep enabled
- **Note:** Theme functionality needed for proper export

#### 3. Simply Static / Simply Static Pro
- **Status:** **Required** - obviously keep enabled

### Plugin Disable Workflow

**Before Export:**
1. Go to WordPress Admin → Plugins
2. Deactivate all plugins listed in "Critical: Plugins to Disable" section
3. Keep only essential plugins active (Divi, Simply Static, Google Reviews if needed)
4. Run Simply Static export
5. Re-enable plugins after export (if needed for WordPress site)

**After Export:**
- Re-enable plugins on WordPress site if needed
- Add Google Analytics directly to static HTML (if analytics needed)
- Replace contact forms with static links or third-party service

### Identifying Problematic Plugins

To identify which plugins are causing issues in your export:

1. **Check HTML for plugin references:**
   - Search for `/wp-content/plugins/` in exported HTML
   - Look for `admin-ajax.php` references
   - Check for unnecessary JavaScript files

2. **Look for broken functionality:**
   - Lazy-loading not working
   - Analytics not tracking
   - Forms not submitting
   - Unnecessary JavaScript errors

3. **Check file size:**
   - Large HTML files may indicate plugin bloat
   - Multiple unnecessary script tags

### Post-Export Plugin Cleanup

After export, you may need to manually remove:

- References to `/wp-admin/admin-ajax.php` in JavaScript
- Broken lazy-loading attributes (convert to standard `src` attributes)
- Unnecessary analytics scripts (replace with direct GA4 code)
- Broken form submissions (replace with mailto links or third-party forms)

---

## Standard Operating Procedure

### Prerequisites

Before beginning the migration, ensure you have:

- WordPress site with Simply Static Pro plugin installed and activated
- WPMUDev account and Hub access (for managing WordPress sites)
- GitHub account (personal or organization)
- Cloudflare access for staging DNS configuration
- Admin access to WordPress site

### Step 1: GitHub Personal Access Token (PAT) Setup

1. **Create GitHub Account** (if needed)
   - Go to https://github.com/join to create an account

2. **Generate Personal Access Token**
   - Log in to GitHub and click your profile picture (top right)
   - Click "Settings" from the dropdown menu
   - Click "Developer settings" in the left sidebar
   - Click "Personal access tokens"
   - Choose token type:
     - **Fine-grained tokens (Recommended):** More secure, can be limited to specific repository
     - **Tokens (classic):** Required for organization accounts in some cases
   - Click "Generate new token" (classic or fine-grained based on account type)

3. **Configure Token Permissions**
   
   **For Fine-grained Tokens (Recommended):**
   - **Token name:** Give it a descriptive name (e.g., "Simply Static - Site Name")
   - **Expiration:** Set appropriate expiration date
   - **Repository access:** Select **"Only select repositories"**
   - **Repository selection:** Choose the specific repository for this export
   - **Permissions:**
     - **Repository permissions → Contents:** Set to **Read and write**
     - **Account permissions → E-Mail address:** Set to **Read-only** (if needed)
   - Click **Generate token**
   
   **For Classic Tokens:**
   - **Name:** Give your token a descriptive name (e.g., "Simply Static - Site Name")
   - **Expiration:** Set appropriate expiration date
   - **Repository Selection:** Select the repository you want to give Simply Static access to
   - **Repository Permissions:**
     - Set "Contents" to **Read & Write**
   - **Account Permissions:**
     - Set "E-Mail address" to **Read-only**
   - Click "Generate Token"
   
   **Best Practice:** Use fine-grained tokens when possible - they're more secure and can be limited to only the specific repository needed.

4. **Copy and Save Token**
   - **Important:** Copy the token immediately - you won't be able to see it again
   - Store securely (password manager recommended)

5. **Organization Approval** (if using org account)
   - If using a GitHub organization account, the PAT requires admin approval
   - Contact your GitHub org admin to approve the token
   - Wait for approval before proceeding

**Reference:** [Simply Static GitHub Integration Guide](https://docs.simplystatic.com/article/33-set-up-the-github-integration)

### Step 2: Create GitHub Repository

1. **Create New Repository**
   - Go to GitHub and create a new repository
   - **Important:** Add a README file when creating the repository
   - Simply Static cannot create the repository - it can only push files to existing repositories

2. **Repository Settings**
   - Choose appropriate visibility (Public or Private)
   - Note the repository name for Simply Static configuration

### Step 3: Configure Simply Static

1. **Navigate to Simply Static Settings**
   - In WordPress admin, go to **Simply Static → Settings → Deployment**
   - Select **"GitHub"** as deployment method

2. **Configure URL Replacement Settings (CRITICAL)**
   
   **For Production Sites (served at specific domain):**
   
   Navigate to **Simply Static → Settings → General → Replacing URLs**
   
   - **Destination URL Type:** Set to **"Absolute URL"**
   - **Static URL:** Enter your production domain (e.g., `https://alltypetowing.com`)
   - **Force URL replacements:** Keep **ON** (enabled)
   
   **Why This Matters:**
   - Absolute URLs ensure all assets (CSS, JS, images) load correctly when served at your domain
   - Prevents path issues when deploying to production
   - Works correctly with CDN and custom domains
   
   **Example Configuration:**
   ```
   Destination URL Type: Absolute URL
   Static URL: https://alltypetowing.com
   Force URL replacements: ✓ ON
   ```
   
   **Note:** For local testing, you may need relative paths, but for production deployment, absolute URLs are recommended.

3. **Configure GitHub Settings**
   - **Account Type:** Select Personal or Organization
   - **GitHub User / Organization:** Enter your GitHub username or organization name
   - **GitHub E-Mail:** Enter your GitHub account email address
   - **Personal Access Token:** Paste the PAT you created earlier
   - **Repository:** Enter the repository name (must already exist)
   - **Visibility:** Select Public or Private (must match repository settings)
   - **Branch Name:** 
     - Default: `main`
     - **For GitHub Pages:** See [GitHub Pages Deployment Requirements](#github-pages-deployment-requirements) below
   - **Webhook URL:** Leave blank (optional - not required for basic deployment)

4. **GitHub Pages Deployment Requirements (IMPORTANT)**
   
   **If deploying to GitHub Pages, note these requirements:**
   
   **Private Repository Limitations:**
   - **GitHub Pages from private repos requires a paid GitHub plan** (GitHub Pro, Team, or Enterprise)
   - **Organization policy** must allow GitHub Pages for private repositories
   - Free accounts cannot use GitHub Pages with private repositories
   
   **GitHub Pages Branch/Folder Requirements:**
   
   GitHub Pages expects one of these configurations:
   
   **Option 1: `gh-pages` Branch (Recommended)**
   - Set **Branch Name** in Simply Static to: `gh-pages`
   - GitHub Pages will serve from the `gh-pages` branch
   - Configure in GitHub: Settings → Pages → Source: `gh-pages` branch
   
   **Option 2: `/docs` Folder on `main` Branch**
   - Set **Branch Name** in Simply Static to: `main`
   - Set **Folder** in Simply Static to: `docs`
   - GitHub Pages will serve from `/docs` folder on `main` branch
   - Configure in GitHub: Settings → Pages → Source: `/docs` folder on `main` branch
   
   **Option 3: Root of `main` Branch (Public Repos Only)**
   - Set **Branch Name** to: `main`
   - Leave **Folder** blank (root)
   - Only works for **public repositories** on free accounts
   - Configure in GitHub: Settings → Pages → Source: `/` (root) on `main` branch
   
   **Current Configuration (All Type Towing):**
   - Repository: Private
   - Branch: `main`
   - Folder: (blank - root)
   - **Issue:** This configuration won't work with GitHub Pages on a private repo
   - **Solution:** Either:
     1. Use `gh-pages` branch (change Branch Name to `gh-pages`)
     2. Use `/docs` folder (set Folder to `docs` and keep Branch as `main`)
     3. Make repository public (if acceptable)
     4. Upgrade to paid GitHub plan (if private repo required)
   
   **Note:** This doesn't affect local testing (`http://localhost:8080/`), but will cause issues when trying to publish via GitHub Pages.

4. **Save Settings**
   - Click "Save Changes"
   - Verify no error messages appear

### Step 4: Pre-Migration Site Preparation

1. **Disable Conflicting Plugins (CRITICAL)**
   - Review [Plugin Management](#plugin-management) section for complete list
   - **Disable Defender Pro** (or whitelist crawler / allow loopback requests) - **CRITICAL**
   - Disable **Hummingbird** plugin (Simply Static will warn if active)
   - Disable **WP Smush Pro** (causes lazy-loading issues in static HTML)
   - Disable **Beehive Analytics / Matomo** (won't work in static HTML)
   - Disable **SmartCrawl SEO** (unnecessary overhead)
   - Disable **Hustle** and other optimization plugins
   - Disable all security, caching, and backup plugins
   - **Why:** Security/hardening plugins often block loopback/"bot-like" requests from Simply Static's crawler
   - Re-enable after migration if needed for WordPress site

2. **Clear Divi Cache (CRITICAL)**
   - Navigate to **Divi → Theme Options → Performance**
   - Click **"Clear"** to clear Divi cache
   - **Why:** Cached content may interfere with export or serve different markup
   - This ensures fresh content is exported

3. **Cloudflare Configuration**
   - Log in to Cloudflare dashboard
   - Navigate to your staging site's DNS settings
   - **Set to Developer Mode** (bypasses cache during migration)
   - Keep in Developer Mode throughout migration process

4. **Verify Staging Site**
   - Ensure staging site is accessible
   - Note the staging DNS configuration for post-migration

### Step 5: Execute Migration

1. **Start Static Site Export**
   - In WordPress admin, go to **Simply Static → Generate**
   - **Important:** Use **"Full Export"** (not single page export)
   - Click **"Start Static Site Export"**
   - Monitor export progress
   - **Note:** Single page exports may miss content if crawler is blocked

2. **Monitor Export**
   - Watch for any errors or warnings
   - Note any CSS validation warnings (see CSS section)
   - Wait for export to complete

3. **Verify Divi Assets Are Included (CRITICAL)**
   
   **Divi Theme Dependencies:**
   
   Divi relies on generated CSS and assets that **must** be included in the export. After export, verify these directories exist in your export folder/repository:
   
   - ✅ `wp-content/` (must exist)
   - ✅ `wp-content/themes/Divi/` (Divi theme files)
   - ✅ `wp-content/uploads/` (images and media)
   - ✅ `wp-content/et-cache/` (**CRITICAL** - Divi generated CSS)
   - ✅ `wp-includes/` (WordPress core assets)
   
   **Why This Matters:**
   - Divi generates CSS files in `wp-content/et-cache/` that are referenced in your HTML
   - If `wp-content/et-cache/` is missing, CSS files won't load → broken layout
   - Simply Static has a Divi integration that should be enabled to ensure these assets are included
   
   **How to Verify:**
   1. Check your export folder (local) or GitHub repository
   2. Navigate to `wp-content/` directory
   3. Confirm `et-cache/` folder exists and contains CSS files
   4. Verify `themes/Divi/` folder exists
   5. Check that `uploads/` folder contains your images
   
   **If Assets Are Missing:**
   - Ensure Simply Static Divi integration is enabled
   - Check Simply Static settings for asset inclusion
   - Re-export if assets are missing
   - See [Troubleshooting: Missing Divi Assets](#missing-divi-assets) section

4. **Verify GitHub Deployment**
   - Go to your GitHub repository
   - Verify files have been pushed successfully
   - Check commit history for new commit from Simply Static
   - **Verify Divi assets are in repository** (check `wp-content/et-cache/` exists)

5. **Check for CSS Issues**
   - Review terminal/IDE for CSS warnings
   - Check for `unknownProperties` errors
   - See [CSS Breaking Changes & Fixes](#css-breaking-changes--fixes) section

### Step 6: Post-Migration Configuration

1. **Cloudflare DNS Setup**
   - Associate staging DNS in Cloudflare with static site
   - Update DNS records if needed
   - Verify DNS propagation

2. **Staging Site Verification**
   - Test staging site URL
   - Verify all pages load correctly
   - Check asset loading

---

## CSS Breaking Changes & Fixes

### Known CSS Issues

#### Issue 1: Invalid `background-repeat-y` Property

**Problem:**
- `background-repeat-y` is not a valid CSS property
- Causes `unknownProperties` linting errors
- Found in Divi theme exports

**Location:**
- Typically found in `.et_pb_bottom_inside_divider` and `.et_pb_top_inside_divider` selectors
- Example: `index.html` line 51 (in minified CSS)

**Fix:**
```css
/* Before (Invalid) */
background-repeat-y: no-repeat;

/* After (Valid) */
background-repeat: no-repeat;
```

**Search/Replace Pattern:**
- Search for: `background-repeat-y:no-repeat` or `background-repeat-y: no-repeat`
- Replace with: `background-repeat:no-repeat` or `background-repeat: no-repeat`

**Note:** The `background-repeat` property controls repetition on both axes. Use `no-repeat` to prevent repetition in both directions.

### CSS Validation Checklist

After migration, complete this checklist:

- [ ] Run CSS linter/validator on exported files
- [ ] Check terminal/IDE for CSS warnings
- [ ] Fix all `unknownProperties` errors
- [ ] Verify no invalid CSS properties remain
- [ ] Test site rendering in browser after fixes
- [ ] Check responsive design (mobile/tablet/desktop)
- [ ] Verify no console errors related to CSS

### Common CSS Issues to Watch For

1. **Invalid Property Names**
   - Examples: `background-repeat-y`, `background-size-x`
   - Fix: Use standard CSS properties

2. **Deprecated Vendor Prefixes**
   - Old: `-webkit-`, `-moz-`, `-ms-` prefixes
   - Fix: Use standard properties or modern prefixes

3. **Missing Semicolons in Minified CSS**
   - Can break CSS parsing
   - Fix: Ensure proper syntax

4. **Invalid Color Values**
   - Malformed hex codes or color names
   - Fix: Use valid CSS color values

5. **Broken Media Queries**
   - Syntax errors in `@media` rules
   - Fix: Verify media query syntax

### How to Find CSS Issues

1. **IDE/Editor Linting**
   - Most modern editors show CSS warnings
   - Look for red/yellow underlines in CSS
   - Check "Problems" panel in VS Code

2. **Terminal Output**
   - Check terminal/console during export
   - Look for CSS validation warnings
   - Note file names and line numbers

3. **Browser DevTools**
   - Open browser console (F12)
   - Check for CSS-related errors
   - Inspect computed styles

4. **CSS Validator Tools**
   - Use online CSS validators
   - W3C CSS Validator: https://jigsaw.w3.org/css-validator/
   - Check exported CSS files

5. **Search for Known Issues**
   - Search codebase for known invalid properties
   - Use grep/search: `background-repeat-y`, `background-size-x`, etc.

---

## Troubleshooting Guide

### Simply Static + Hummingbird Conflict

**Problem:**
Simply Static warns about Hummingbird plugin being active.

**Solution:**
1. Go to WordPress admin → Plugins
2. Deactivate Hummingbird plugin
3. Run Simply Static export
4. Re-enable Hummingbird after migration (if needed for WordPress site)

**Prevention:**
- Always disable Hummingbird before running exports
- Add to pre-migration checklist

### Plugin-Related Issues in Static Export

**Problem:**
- Broken lazy-loading on images
- Unnecessary JavaScript files in export
- References to `/wp-admin/admin-ajax.php` in code
- Analytics not working
- Large file sizes due to plugin bloat

**Solution:**
1. Review [Plugin Management](#plugin-management) section
2. Disable problematic plugins before export:
   - WP Smush Pro (lazy-loading issues)
   - Beehive Analytics / Matomo (analytics bloat)
   - SmartCrawl SEO (unnecessary overhead)
   - Security, caching, and backup plugins
3. Re-export after disabling plugins
4. Manually add Google Analytics to static HTML if needed
5. Replace broken lazy-loading with standard image tags if needed

**Prevention:**
- Always review plugin list before export
- Keep plugin disable checklist updated
- Test export with minimal plugins first

### Cloudflare Caching Issues

**Problem:**
Old cached content showing after migration, or changes not appearing.

**Solution:**
1. Set Cloudflare to **Developer Mode** before migration
2. Keep Developer Mode active during entire migration process
3. Clear Cloudflare cache after migration
4. Wait for cache to refresh

**Prevention:**
- Always use Developer Mode during migration
- Document Cloudflare settings in migration notes

### CSS/JavaScript Not Loading (Local Testing)

**Problem:**
- CSS files fail to load
- JavaScript doesn't execute
- Site appears unstyled
- Browser console shows `ERR_FILE_NOT_FOUND` errors
- Paths like `file:///C:/wp-content/...` in errors

**Root Cause:**
Opening `index.html` directly by double-clicking uses `file://` protocol, which browsers restrict for security (CORS). This prevents CSS and JavaScript from loading properly.

**Solution:**
1. **Never double-click `index.html`** - Always use a local web server
2. **Use local web server instead:**
   - Windows PowerShell: `py -m http.server 8080`
   - Then open: `http://localhost:8080/`
3. **Verify paths are relative** (not absolute like `/wp-content/...`)
4. **Test via HTTP server** before deploying

**Prevention:**
- Always test exports via local web server
- Document this in migration checklist
- Add to team training materials

**See Also:** [Testing the Static Export Locally](#testing-the-static-export-locally) section above

### GitHub Deployment Failures

**Problem:**
PAT not working, access denied, or deployment fails.

**Solutions:**

1. **Verify PAT Permissions**
   - Check PAT has "Contents" set to Read & Write
   - Verify "E-Mail address" is Read-only
   - Ensure PAT hasn't expired

2. **Check Organization Approval**
   - If using org account, verify PAT is approved by admin
   - Contact GitHub org admin if needed

3. **Verify Repository Access**
   - Ensure repository exists
   - Check PAT has access to correct repository
   - Verify repository name matches Simply Static settings

4. **Check Repository Settings**
   - Ensure repository visibility matches Simply Static settings
   - Verify branch name is correct (main vs gh-pages)

5. **Regenerate PAT (Step-by-Step)**
   
   If issues persist, revoke the old token and create a new one:
   
   **Step 1: Revoke Old Token**
   1. Go to GitHub (logged in as the account that created the token)
   2. Click your **profile picture** (top right) → **Settings**
   3. In left sidebar, click **Developer settings**
   4. Click **Personal access tokens**
   5. Choose token type:
      - **Classic token:** Click **Tokens (classic)**
      - **Fine-grained token:** Click **Fine-grained tokens**
   6. Find the token that matches what you used for Simply Static
   7. Click **Revoke** (or **Delete**)
   
   **Step 2: Generate New Token**
   1. Click **Generate new token** (classic or fine-grained based on account type)
   2. **For Fine-grained tokens (Recommended):**
      - **Token name:** Give it a descriptive name (e.g., "Simply Static - All Type Towing")
      - **Expiration:** Set appropriate expiration date
      - **Repository access:** Select **"Only select repositories"**
      - **Repository selection:** Choose the specific repository for this export
      - **Permissions:**
        - **Repository permissions → Contents:** Set to **Read and write**
        - **Account permissions → E-Mail address:** Set to **Read-only** (if needed)
      - Click **Generate token**
   
   3. **For Classic tokens:**
      - **Note:** Give your token a descriptive name (e.g., "Simply Static - All Type Towing")
      - **Expiration:** Set appropriate expiration date
      - **Repository Selection:** Select the repository you want to give Simply Static access to
      - **Repository Permissions:**
        - Set **Contents** to **Read & Write**
      - **Account Permissions:**
        - Set **E-Mail address** to **Read-only** (if needed)
      - Click **Generate Token**
   
   **Step 3: Copy and Use New Token**
   1. **Important:** Copy the token immediately - you won't be able to see it again
   2. Store securely (password manager recommended)
   3. Go to WordPress admin → **Simply Static → Settings → Deployment**
   4. Paste the new token into **Personal Access Token** field
   5. Click **Save Changes**
   6. Test export to verify token works
   
   **Best Practice:**
   - Use **Fine-grained tokens** when possible (more secure)
   - Limit token to **only the specific repository** needed
   - Set minimum permissions required (usually just Contents: Read and write)
   - Use descriptive token names for easy identification

### GitHub Pages Deployment Issues

**Problem:**
- GitHub Pages not serving site
- "Page build failed" errors
- Site not accessible after deployment
- Configuration seems correct but Pages won't activate

**Root Causes:**

1. **Private Repository Limitations:**
   - **GitHub Pages from private repos requires a paid GitHub plan** (Pro, Team, or Enterprise)
   - Free accounts cannot use GitHub Pages with private repositories
   - Organization policy may not allow GitHub Pages for private repos

2. **Incorrect Branch/Folder Configuration:**
   - GitHub Pages expects either:
     - `gh-pages` branch, OR
     - `/docs` folder on `main` branch
   - Root of `main` branch only works for **public repositories**

**Solutions:**

1. **For Private Repositories:**
   - **Option A:** Upgrade to paid GitHub plan (Pro, Team, or Enterprise)
   - **Option B:** Make repository public (if acceptable)
   - **Option C:** Use alternative hosting (Netlify, Cloudflare Pages, etc.)

2. **Fix Branch/Folder Configuration:**
   
   **If using `main` branch with root folder (current config):**
   - This only works for **public repositories**
   - For private repos, you must use one of these:
   
   **Option 1: Switch to `gh-pages` Branch (Recommended)**
   - In Simply Static: Change **Branch Name** to `gh-pages`
   - Leave **Folder** blank
   - In GitHub: Settings → Pages → Source: `gh-pages` branch
   - Re-export to push to `gh-pages` branch
   
   **Option 2: Use `/docs` Folder on `main` Branch**
   - In Simply Static: Keep **Branch Name** as `main`
   - Set **Folder** to `docs`
   - In GitHub: Settings → Pages → Source: `/docs` folder on `main` branch
   - Re-export to push to `/docs` folder
   
   **Option 3: Make Repository Public**
   - Change repository visibility to Public
   - Keep current configuration (root of `main` branch)
   - Configure GitHub Pages: Settings → Pages → Source: `/` (root) on `main` branch

3. **Verify GitHub Pages Settings:**
   - Go to repository → Settings → Pages
   - Check "Source" is set correctly:
     - `gh-pages` branch (if using Option 1)
     - `/docs` folder on `main` branch (if using Option 2)
     - `/` (root) on `main` branch (if public repo, Option 3)
   - Verify custom domain settings if applicable

4. **Check for Build Errors:**
   - Go to repository → Actions tab
   - Check for failed GitHub Pages builds
   - Review error messages
   - Common issues: Jekyll processing errors, invalid HTML, etc.

**Prevention:**
- Plan GitHub Pages configuration before export
- Verify repository visibility and plan limitations
- Choose appropriate branch/folder structure
- Test deployment configuration early

**Current Configuration (All Type Towing):**
- Repository: **Private**
- Branch: `main`
- Folder: (blank - root)
- **Status:** ⚠️ **This configuration won't work with GitHub Pages**
- **Required Action:** Either switch to `gh-pages` branch, use `/docs` folder, make repo public, or upgrade to paid plan

**See Also:** [GitHub Pages Deployment Requirements](#github-pages-deployment-requirements-important) section above

### Missing Divi Assets

**Problem:**
- Site loads but appears unstyled or broken
- CSS files return 404 errors
- Layout is completely broken
- Browser console shows errors like: `Failed to load resource: net::ERR_FILE_NOT_FOUND` for CSS files
- Paths like `/wp-content/et-cache/2/et-divi-dynamic-*.css` fail to load

**Root Cause:**
Divi theme generates CSS files in `wp-content/et-cache/` that are critical for styling. If this directory is missing from the export, all CSS references will fail.

**Solution:**

1. **Verify Divi Integration is Enabled**
   - In Simply Static settings, ensure Divi integration is enabled
   - This ensures Divi assets are included in export

2. **Check Export Folder/Repository**
   - Verify these directories exist:
     - ✅ `wp-content/et-cache/` (**CRITICAL** - must exist)
     - ✅ `wp-content/themes/Divi/`
     - ✅ `wp-content/uploads/`
     - ✅ `wp-includes/`

3. **If Assets Are Missing:**
   - Re-export the site with Divi integration enabled
   - Check Simply Static settings for asset inclusion options
   - Verify export completed successfully (no errors)
   - Check export log for any skipped files

4. **Verify CSS Files Exist:**
   - Navigate to `wp-content/et-cache/2/` (or similar numbered folder)
   - Confirm CSS files like `et-divi-dynamic-*.css` are present
   - Check file sizes are reasonable (not 0 bytes)

**Prevention:**
- Always verify Divi assets after export
- Enable Simply Static Divi integration
- Add asset verification to post-export checklist
- Test site rendering immediately after export

**See Also:** [Step 5: Verify Divi Assets Are Included](#step-5-execute-migration) section above

### Security/Caching Plugins Blocking Crawler

**Problem:**
- Export appears to complete but content is missing
- Main content sections not included in export
- HTML structure incomplete (missing sections)
- Export seems successful but site looks broken
- Different content than expected in export

**Root Cause:**
Security and caching plugins (especially Defender Pro, but also others) commonly:
- **Block loopback / "bot-like" requests** from Simply Static's crawler
- **Serve different markup** to non-logged-in users or certain user agents
- **Inject optimizations** that rely on runtime JavaScript
- **Cache mutations** that interfere with crawler

This makes it appear that Simply Static "missed" your content, when actually the crawler was blocked or received different content.

**Fast Isolation Test (No Guessing):**

1. **Temporarily disable Defender Pro** (or whitelist crawler / allow loopback requests)
2. **Clear Divi cache:**
   - Navigate to **Divi → Theme Options → Performance**
   - Click **"Clear"** to clear Divi cache
3. **Run a fresh full export** (not single page export)
4. **Preview via local web server:**
   - `py -m http.server 8080`
   - Open `http://localhost:8080/`
5. **Check if content is now present**

**If that fixes it:** Then you know it's crawler-blocking or caching mutation, not Simply Static "missing your content."

**Solution:**

1. **Disable Security Plugins Before Export:**
   - **Defender Pro** - Disable completely (or whitelist crawler)
   - **Wordfence** - Disable or whitelist
   - **iThemes Security** - Disable or configure to allow loopback
   - **Other security plugins** - Disable during export

2. **Disable Caching Plugins:**
   - **Hummingbird** - Disable (Simply Static will warn)
   - **WP Super Cache** - Disable
   - **W3 Total Cache** - Disable
   - **Other caching plugins** - Disable during export

3. **Clear All Caches:**
   - **Divi Cache:** Divi → Theme Options → Performance → Clear
   - **WordPress Cache:** Clear via caching plugin (if enabled)
   - **Browser Cache:** Clear browser cache or use incognito mode

4. **Alternative: Whitelist Crawler (if disabling not possible):**
   - In Defender Pro: Whitelist Simply Static's user agent
   - Allow loopback requests from localhost
   - Configure to serve same markup to crawler

5. **Run Fresh Export:**
   - Use **full export** (not single page)
   - Monitor export for any errors
   - Verify content is included

**Prevention:**
- Always disable security/caching plugins before export
- Clear Divi cache before every export
- Add to pre-migration checklist
- Document which plugins were disabled
- Re-enable plugins after successful export

**Common Offenders:**
- **Defender Pro** - Very common cause
- **WP Smush** - Can interfere with crawler
- **Hustle** - Optimization plugin that may block crawler
- **Security plugins** - Often block "bot-like" requests
- **Caching plugins** - May serve cached/different content

**See Also:** 
- [Step 4: Pre-Migration Site Preparation](#step-4-pre-migration-site-preparation)
- [Plugin Management: Security Plugins](#plugin-management)

### CSS Breaking Changes

**Problem:**
CSS validation errors in exported files, styling broken.

**Solution:**
1. Review [CSS Breaking Changes & Fixes](#css-breaking-changes--fixes) section
2. Check terminal/IDE for specific CSS warnings
3. Use search/replace to fix invalid properties
4. Run CSS validator on fixed files
5. Test site rendering in browser

**Prevention:**
- Review CSS before export
- Fix known issues proactively
- Keep list of common CSS issues updated

### Asset Path Issues

**Problem:**
Images, CSS, or JavaScript files not loading.

**Solutions:**

1. **Check URL Settings in Simply Static**
   - Navigate to **Simply Static → Settings → General → Replacing URLs**
   - **For Production Sites (recommended):**
     - **Destination URL Type:** Set to **"Absolute URL"**
     - **Static URL:** Enter your production domain (e.g., `https://alltypetowing.com`)
     - **Force URL replacements:** Keep **ON** (enabled)
     - This ensures all assets load correctly when served at your domain
   - **For Local Testing:**
     - Absolute URLs work fine when testing via local web server (`http://localhost:8080`)
     - Relative paths may be needed only if testing via `file://` protocol (not recommended)
   - Check "Additional URLs" if needed
   
   **Important:** Always use absolute URLs for production deployments. The path conversion we did earlier was a workaround for an incorrectly configured export. The proper solution is to configure Simply Static correctly from the start.

2. **Verify Divi Assets Export (CRITICAL)**
   - **Check `wp-content/et-cache/` directory exists** - Contains Divi generated CSS (CRITICAL)
   - Check `wp-content/themes/Divi/` directory - Divi theme files
   - Check `wp-content/uploads/` directory - Images and media
   - Check `wp-includes/` directory - WordPress core assets
   - Verify Simply Static Divi integration is enabled
   - See [Missing Divi Assets](#missing-divi-assets) troubleshooting section if assets are missing

3. **Check File Paths**
   - Verify paths are correct in HTML
   - Check for broken relative paths
   - Ensure base URL is correct

4. **GitHub Repository Structure**
   - Verify files are in correct directories
   - Check for missing files in repository

### WPMUDev Hub Integration

**Problem:**
Hub pages or functionality not working in static export.

**Notes:**
- Hub pages may need special handling in static export
- Some Hub functionality may require WordPress backend
- Verify Hub requirements before migration

**Solution:**
- Test Hub functionality after migration
- Document any Hub-specific issues
- Consider Hub alternatives if needed

### Export Timeout or Memory Issues

**Problem:**
Export fails due to timeout or memory limits.

**Solutions:**
1. Increase PHP memory limit in WordPress
2. Increase max execution time
3. Export in smaller batches if possible
4. Contact hosting provider for limits

---

## Post-Migration Verification

Complete this verification checklist after migration:

### Testing the Static Export Locally

**⚠️ IMPORTANT: Never test by double-clicking `index.html`**

Opening HTML files directly via `file://` protocol can cause CSS and JavaScript to fail loading due to browser security restrictions (CORS). Always use a local web server.

#### Windows PowerShell

1. Open PowerShell in the export folder
2. Run the following command:
   ```powershell
   py -m http.server 8080
   ```
3. Open your browser and navigate to:
   ```
   http://localhost:8080/
   ```

#### Alternative: Python 3 (if `py` doesn't work)

```powershell
python -m http.server 8080
```

#### Alternative: Node.js (if Python not available)

```powershell
npx http-server -p 8080
```

#### Why This Matters

- **`file://` protocol:** Browsers restrict loading local files, causing CSS/JS to fail
- **HTTP server:** Mimics real web hosting, allows proper resource loading
- **Testing:** Ensures export works correctly before deployment

**Note:** The server will run until you press `Ctrl+C` to stop it.

### Site Functionality

- [ ] **All pages load correctly**
  - Homepage loads
  - All navigation links work
  - No 404 errors

- [ ] **CSS styling is intact**
  - Layout looks correct
  - Colors and fonts display properly
  - No broken styling
  - **Tested via local web server (not `file://` protocol)**

- [ ] **Responsive design works**
  - Test on mobile devices
  - Test on tablet
  - Test on desktop
  - Verify breakpoints work

- [ ] **Divi assets are present (CRITICAL)**
  - `wp-content/et-cache/` directory exists (contains Divi generated CSS)
  - `wp-content/themes/Divi/` directory exists
  - `wp-content/uploads/` directory exists (images)
  - `wp-includes/` directory exists
  - Simply Static Divi integration is enabled

- [ ] **Images and assets load**
  - All images display
  - CSS files load (especially from `wp-content/et-cache/`)
  - JavaScript files load
  - No broken asset links
  - No 404 errors for CSS files in browser console

- [ ] **Navigation menus work**
  - All menu items clickable
  - Dropdown menus function (if applicable)
  - Mobile menu works

- [ ] **Forms functionality** (if applicable)
  - Note: Forms may require backend
  - Test form submission
  - Verify form styling

- [ ] **Google Reviews widget displays**
  - Widget loads correctly
  - Reviews show properly
  - Styling is intact

### Technical Verification

- [ ] **Cloudflare DNS is pointing correctly**
  - Verify DNS records
  - Check DNS propagation
  - Test staging site URL

- [ ] **No console errors**
  - Open browser DevTools
  - Check Console tab
  - Fix any JavaScript errors

- [ ] **CSS validation passes**
  - Run CSS validator
  - Fix any remaining CSS issues
  - Verify no `unknownProperties` errors

- [ ] **GitHub repository is up to date**
  - Check latest commit
  - Verify all files are present
  - Check file sizes are reasonable

- [ ] **Performance check**
  - Test page load speed
  - Check image optimization
  - Verify lazy loading works (if applicable)

### Documentation

- [ ] **Update migration notes**
  - Document any issues encountered
  - Note CSS fixes applied
  - Record Cloudflare settings

- [ ] **Update team documentation**
  - Share migration completion
  - Note any special considerations
  - Document post-migration steps

---

## Additional Notes

### Webhook Configuration (Optional)

Simply Static supports webhook URLs for notifying external services after deployment. This is **optional** and not required for basic GitHub deployment.

**When to use webhooks:**
- Automated deployment to additional services (Netlify, Vercel, etc.)
- GitHub Actions workflows
- Custom deployment pipelines

**Configuration:**
- Leave blank for basic GitHub deployment
- Add webhook URL if using GitHub Actions or other automation

### Staging vs Production

- Always test migration on **staging site first**
- Verify staging site works before production migration
- Document staging DNS configuration
- Keep staging and production configurations separate

### Version Control

- GitHub repository provides version control automatically
- Each Simply Static export creates a new commit
- Use Git tags for important versions
- Document major changes in commit messages

---

## References

- **Simply Static GitHub Integration Guide:** https://docs.simplystatic.com/article/33-set-up-the-github-integration
- **Current CSS Issue Fixed:** `background-repeat-y` → `background-repeat` in `index.html` line 51
- **W3C CSS Validator:** https://jigsaw.w3.org/css-validator/

---

## Revision History

- **2025-01-27:** Added Plugin Management section with detailed list of plugins to disable before export (WP Smush Pro, Beehive Analytics, SmartCrawl SEO, security/caching/backup plugins). Updated troubleshooting guide with plugin-related issues. Added plugin disable workflow and post-export cleanup procedures.
- **Initial Version:** Documented migration process including Simply Static Pro setup, GitHub PAT configuration, CSS fix procedures, and troubleshooting guide.

---

**Last Updated:** 2025-01-27

