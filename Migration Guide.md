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

- [ ] **Disable Hummingbird plugin** (Simply Static conflicts - Simply Static will warn if active) - CRITICAL
- [ ] **Disable problematic plugins** (see [Plugin Management](#plugin-management) section below) - only if conflicts occur
- [ ] **Clear Divi cache** (Divi → Theme Options → Performance → Clear) - Optional, only for troubleshooting if export issues occur
- [ ] **Run Simply Static Diagnostics** (Simply Static → Diagnostics) - must pass all checks
- [ ] **Verify all diagnostic checks pass** - do not proceed if critical checks fail
- [ ] **Set Cloudflare to Developer Mode** (disable caching during migration)
- [ ] **Verify Simply Static Pro is installed and activated**
- [ ] **Enable Simply Static Divi integration** (ensures Divi assets are included)
- [ ] **Create GitHub Personal Access Token (PAT)** (see Step 1 in SOP)
- [ ] **Get GitHub org admin approval for PAT** (if using organization account)
- [ ] **Create GitHub repository with README** (Simply Static needs repository to exist first)
- [ ] **Plan GitHub Pages configuration** (can run from root of `main` branch - no need for `gh-pages` branch)
- [ ] **Configure Simply Static file path settings** (General → Replacing URLs: Absolute URL for production, Relative Path for local testing)
- [ ] **Configure Simply Static GitHub deployment settings** (branch: `main`, folder: root)
- [ ] **Set up local web server for testing** (see Step 6 in SOP)
- [ ] **Test static export via local web server** (use `python -m http.server 8081`, not double-click) - test after export
- [ ] **Plan Cloudflare staging site setup** (DNS, SSL, caching configuration)

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

#### 4. Security Plugins (Only if Conflicts Occur)
- **Examples:** Defender Pro, Wordfence, iThemes Security, WP Defender
- **Problem:** 
  - May block loopback / "bot-like" requests from Simply Static's crawler
  - May serve different markup to non-logged-in users / certain user agents
  - May inject optimizations that rely on runtime JavaScript
  - Can cause incomplete exports or missing content if conflicts occur
- **Action:** **Only disable if you encounter conflicts or issues** (or whitelist crawler / allow loopback requests)
- **Note:** Not all security plugins cause issues - only disable if export problems occur

#### 5. Caching Plugins
- **Examples:** WP Super Cache, W3 Total Cache, Hummingbird
- **Problem:** Not needed - static export is already "cached"
- **Action:** **Disable before export**

#### 6. Backup Plugins
- **Examples:** Snapshot Backups, UpdraftPlus
- **Problem:** Generally no conflicts with static export
- **Action:** **Can be left enabled** - only disable if conflicts occur

#### 7. Contact Form Plugins
- **Examples:** Forminator, Contact Form 7, Gravity Forms
- **Problem:** Forms won't work in static HTML (no backend processing) - but no conflicts with export
- **Action:** **Can be left enabled** - forms will need to be rebuilt/replaced with custom code or third-party form service in the static site

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
   
   **Simplest Configuration (Recommended):**
   - Set **Branch Name** in Simply Static to: `main`
   - Leave **Folder** blank (root)
   - Configure in GitHub: Settings → Pages → Source: `/` (root) on `main` branch
   - **Note:** This works for both public and private repositories (private repos require paid GitHub plan)
   - **No need for `gh-pages` branch or `/docs` folder** - you can run directly from root
   
   **Alternative Options (If Needed):**
   
   **Option 1: `gh-pages` Branch**
   - Set **Branch Name** in Simply Static to: `gh-pages`
   - GitHub Pages will serve from the `gh-pages` branch
   - Configure in GitHub: Settings → Pages → Source: `gh-pages` branch
   
   **Option 2: `/docs` Folder on `main` Branch**
   - Set **Branch Name** in Simply Static to: `main`
   - Set **Folder** in Simply Static to: `docs`
   - GitHub Pages will serve from `/docs` folder on `main` branch
   - Configure in GitHub: Settings → Pages → Source: `/docs` folder on `main` branch
   
   **Private Repository Limitations:**
   - **GitHub Pages from private repos requires a paid GitHub plan** (GitHub Pro, Team, or Enterprise)
   - Organization policy must allow GitHub Pages for private repositories
   - Free accounts cannot use GitHub Pages with private repositories

4. **Save Settings**
   - Click "Save Changes"
   - Verify no error messages appear

### Step 4: Pre-Migration Site Preparation

1. **Disable Hummingbird Plugin (CRITICAL)**
   - Navigate to **WordPress Admin → Plugins**
   - Find **Hummingbird** plugin
   - Click **"Deactivate"**
   - **Why:** Simply Static conflicts with Hummingbird - Simply Static will warn if active
   - **Note:** Simply Static will display a warning if Hummingbird is still active during export
   - Re-enable after migration if needed for WordPress site

2. **Disable Conflicting Plugins (Only if Issues Occur)**
   - Review [Plugin Management](#plugin-management) section for complete list
   - **Only disable plugins if you encounter conflicts or export issues**
   - Common conflicts:
     - **WP Smush Pro** (causes lazy-loading issues in static HTML)
     - **Beehive Analytics / Matomo** (won't work in static HTML)
     - **SmartCrawl SEO** (unnecessary overhead)
     - **Hustle** and other optimization plugins
   - **Security plugins:** Only disable if crawler is blocked or export fails
   - **Backup plugins:** Generally safe to leave enabled
   - **Contact form plugins:** Safe to leave enabled (forms will need to be rebuilt in static site)
   - Re-enable after migration if needed for WordPress site

3. **Clear Divi Cache (Optional - Troubleshooting Only)**
   - **Not required for normal exports**
   - Only clear if you encounter export issues or stale content
   - Navigate to **Divi → Theme Options → Performance**
   - Click **"Clear"** to clear Divi cache
   - **When to use:** If export shows old/cached content or missing sections

4. **Run Simply Static Diagnostics**
   - Navigate to **Simply Static → Diagnostics**
   - Review all diagnostic checks
   - **Critical Checks:**
     - ✅ **PHP Version:** Should be 7.4 or higher
     - ✅ **cURL Extension:** Must be enabled
     - ✅ **Memory Limit:** Should be at least 256M
     - ✅ **Max Execution Time:** Should be sufficient for large sites
     - ✅ **File Permissions:** Write permissions for export directory
     - ✅ **Plugin Conflicts:** No active conflicting plugins (especially Hummingbird)
   - **Fix any failed checks before proceeding**
   - **Do not proceed with export if diagnostics show critical failures**

5. **Verify All Diagnostic Checks Pass**
   - All critical checks must show ✅ (green/passed)
   - Address any warnings or errors before export
   - Common issues:
     - **Plugin conflicts:** Disable conflicting plugins (see Step 4.1-4.2)
     - **Memory limit too low:** Increase PHP memory limit
     - **File permissions:** Fix directory permissions
   - **Only proceed when all critical diagnostics pass**

6. **Cloudflare Configuration**
   - Log in to Cloudflare dashboard
   - Navigate to your staging site's DNS settings
   - **Set to Developer Mode** (bypasses cache during migration)
   - Keep in Developer Mode throughout migration process

7. **Verify Staging Site**
   - Ensure staging site is accessible
   - Note the staging DNS configuration for post-migration

### Step 5: Configure File Paths (CRITICAL)

**Before exporting, ensure file paths are configured correctly:**

1. **Navigate to Simply Static Settings**
   - Go to **Simply Static → Settings → General → Replacing URLs**

2. **Configure Path Settings**
   
   **For Production Deployment (Recommended):**
   - **Destination URL Type:** Set to **"Absolute URL"**
   - **Static URL:** Enter your production domain (e.g., `https://alltypetowing.com`)
   - **Force URL replacements:** Keep **ON** (enabled)
   - **Why:** Absolute URLs ensure all assets load correctly when served at your domain
   
   **For Local Testing (Alternative):**
   - **Destination URL Type:** Set to **"Relative Path"**
   - **PATH:** Leave blank (or enter `/` for root)
   - **Force URL replacements:** Keep **ON** (enabled)
   - **Note:** Relative paths work for local testing but may cause issues in production
   - **Recommendation:** Use Absolute URLs for production, test via local HTTP server

3. **Verify Path Configuration**
   - Double-check settings match your deployment target
   - For GitHub Pages deployment, use **Absolute URL** with your production domain
   - Save settings before proceeding

**Important:** Simply Static's "Relative Path" setting doesn't convert all paths (inline CSS, data attributes, JavaScript). For production, always use Absolute URLs. See [Troubleshooting: Asset Path Issues](#asset-path-issues) if paths are incorrect.

### Step 6: Execute Migration

1. **Export to GitHub Repository (Primary Option)**
   
   - In WordPress admin, go to **Simply Static → Generate**
   - **Export Type:** Select **"GitHub"** deployment
   - **Verify GitHub settings are configured** (from Step 3)
   - **Important:** Use **"Full Export"** (not single page export)
   - Click **"Start Static Site Export"**
   - Monitor export progress
   - Wait for export to complete

2. **Alternative: Export to Local ZIP (If Needed)**
   
   - **Use this option if you prefer to test locally first or if GitHub export fails**
   - In WordPress admin, go to **Simply Static → Generate**
   - **Export Type:** Select **"ZIP Archive"**
   - **Important:** Use **"Full Export"** (not single page export)
   - Click **"Start Static Site Export"**
   - Monitor export progress
   - Wait for export to complete
   - Download the ZIP file when ready
   - Extract ZIP to a folder for testing

3. **Set Up Local Web Server for Testing**
   
   **Why:** Always test exports via HTTP server, never open HTML files directly (`file://` protocol)
   
   **If you exported to GitHub:**
   - Clone or pull the repository locally
   - Navigate to the repository folder
   
   **If you exported to ZIP:**
   - Extract ZIP to a folder
   - Navigate to extracted folder
   
   **Python HTTP Server (Recommended):**
   ```powershell
   # Navigate to export folder (GitHub repo or extracted ZIP)
   cd C:\path\to\export\folder
   
   # Start HTTP server
   python -m http.server 8081
   ```
   
   **Access your site:**
   - Open browser to: **http://localhost:8081**
   - Or: **http://127.0.0.1:8081**
   
   **Alternative Options:**
   - **Node.js:** `npx http-server -p 8081`
   - **PHP:** `php -S localhost:8081`
   
   **See:** `ROOT_CAUSE_ANALYSIS.md` → "Local Testing with HTTP Server" section for detailed instructions and troubleshooting

4. **Test Export via Local Web Server**
   - Open browser to `http://localhost:8081`
   - **Check for issues:**
     - ✅ All CSS files load (check Network tab in DevTools)
     - ✅ All JavaScript files load
     - ✅ Images display correctly
     - ✅ No console errors (F12 → Console tab)
     - ✅ All content sections are present
     - ✅ Navigation works
   - **Verify file paths:**
     - Check that paths are correct (absolute URLs should work via HTTP server)
     - Verify no `ERR_FILE_NOT_FOUND` errors in console
   - **If problems found:** See troubleshooting steps below

5. **Fix Issues (If Any)**
   
   **If export has problems:**
   - **Missing content sections:** Re-check diagnostics, disable conflicting plugins if needed
   - **Broken CSS/JS paths:** Verify Simply Static path settings (Step 5)
   - **Missing assets:** Ensure Divi integration is enabled, check `wp-content/et-cache/` exists
   - **Plugin code in export:** Disable problematic plugins (Step 4.2)
   - **Make necessary fixes in WordPress**
   - **Re-run export** (to GitHub or ZIP)
   - **Re-test via local web server**
   - **Repeat until export is clean**

6. **Verify GitHub Deployment** (If exported to GitHub)
   - Go to your GitHub repository
   - Verify files have been pushed successfully
   - Check commit history for new commit from Simply Static
   - **Verify Divi assets are in repository:**
     - ✅ Check `wp-content/et-cache/` exists
     - ✅ Check `wp-content/themes/Divi/` exists
     - ✅ Check `wp-content/uploads/` exists
     - ✅ Check `wp-includes/` exists

7. **Verify File Paths in Export**
   - Open a few HTML files (in GitHub or extracted ZIP)
   - Search for path references (e.g., `/wp-content/` or full URLs)
   - **For Absolute URLs:** Verify paths point to your production/staging domain
   - **For Relative Paths:** Verify paths are relative (no leading `/`)
   - **If paths are incorrect:** Re-check Simply Static settings (Step 5) and re-export

8. **Check for CSS Issues**
   - Review exported HTML for CSS warnings
   - Check for `unknownProperties` errors
   - See [CSS Breaking Changes & Fixes](#css-breaking-changes--fixes) section

### Step 7: Post-Migration Configuration

1. **Set Up Staging Site via Cloudflare**
   
   **Configure Cloudflare DNS for Staging:**
   
   - Log in to Cloudflare dashboard
   - Navigate to your domain's DNS settings
   - **Update the `staging` DNS record:**
     - **Type:** CNAME
     - **Name:** `staging`
     - **Content/Target:** `freeforcharity.github.io` (or your organization's GitHub Pages hostname)
     - **Proxy status:** Proxied (orange cloud)
     - **TTL:** Auto
   - **SSL/TLS Settings:**
     - Set SSL/TLS encryption mode to **"Full"** or **"Full (strict)"**
     - Enable **"Always Use HTTPS"** redirect
   - **Cache Settings:**
     - Configure caching rules for static assets
     - Set appropriate cache headers
   - **Page Rules (Optional):**
     - Create page rules for specific paths if needed
     - Configure redirects if necessary

2. **Verify Staging Site**
   - Test `staging.alltypetowing.com` in browser
   - **Verify:**
     - ✅ All pages load correctly
     - ✅ CSS and JavaScript load
     - ✅ Images display
     - ✅ No console errors
     - ✅ SSL certificate is valid
     - ✅ HTTPS redirects work
   - **Check DNS propagation:**
     - Use `nslookup` or online DNS checker
     - Verify DNS records point correctly
   - **Test from multiple locations/devices**

3. **Final Production Migration (When Ready)**
   
   **Once staging site is verified and ready for production:**
   
   - **Update Simply Static Static URL:**
     - Change from `https://staging.alltypetowing.com` to `https://alltypetowing.com`
     - Re-export to GitHub
   
   - **Update GitHub Pages Custom Domain:**
     - Remove `staging.alltypetowing.com` custom domain
     - Add `alltypetowing.com` as custom domain
     - Update CNAME file in repository if needed
   
   - **Update Cloudflare DNS for Production:**
     - Change the `alltypetowing.com` A record from WordPress server IP to GitHub Pages
     - **Type:** CNAME (recommended) or A records
     - **Name:** `alltypetowing.com` (or `@`)
     - **Content/Target:** `freeforcharity.github.io` (or GitHub Pages IPs)
     - **Proxy status:** Proxied
   
   - **Verify Production Site:**
     - Test `alltypetowing.com` loads from GitHub Pages
     - Verify SSL certificate
     - Test all functionality
     - Monitor for any issues
   
   **Migration Complete:** Your WordPress site is now fully migrated to static GitHub Pages!

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

- **2025-01-27:** Major workflow and documentation updates:
  - Reorganized Standard Operating Procedure with proper step-by-step workflow
  - Added explicit Hummingbird disable step (Step 4.1)
  - Added Simply Static Diagnostics step (Step 4.4-4.5) - must pass all checks before export
  - Added file path configuration step (Step 5) - ensure paths are configured correctly
  - Updated export workflow (Step 6):
    - Export to GitHub repository as primary option
    - Export to local ZIP as alternative option
    - Both options should be tested with local web server
    - Comprehensive testing checklist
  - Updated plugin management section:
    - Divi cache clearing: Changed from CRITICAL to optional/troubleshooting only
    - Security plugins: Only disable if conflicts occur (not required)
    - Backup plugins: Can be left enabled unless conflicts
    - Contact form plugins: Can be left enabled (no conflicts, just need to rebuild forms)
  - Updated GitHub Pages configuration:
    - Clarified that you can run directly from root of `main` branch (no need for `gh-pages`)
    - Simplified deployment options
  - Updated Cloudflare staging setup (Step 7):
    - Detailed DNS configuration for staging subdomain
    - Added final production migration steps
    - Clear workflow for moving from staging to production
  - Added comprehensive local testing instructions with HTTP server setup
  - Added troubleshooting workflow for failed exports
- **2025-01-27:** Added Plugin Management section with detailed list of plugins to disable before export (WP Smush Pro, Beehive Analytics, SmartCrawl SEO, security/caching/backup plugins). Updated troubleshooting guide with plugin-related issues. Added plugin disable workflow and post-export cleanup procedures.
- **Initial Version:** Documented migration process including Simply Static Pro setup, GitHub PAT configuration, CSS fix procedures, and troubleshooting guide.

---

**Last Updated:** 2025-01-27

