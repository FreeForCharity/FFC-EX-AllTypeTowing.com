# All Type Towing Website - Site Overview

## What This Is

This is a **static HTML export** of a WordPress website for **All Type Towing**, a towing service business based in Middle River, Maryland. The original WordPress site was built using the **Divi theme** and has been exported as static HTML files for hosting without requiring a WordPress backend.

## Business Information

- **Business Name**: All Type Towing & Transport
- **Phone**: (443) 740-0423
- **Location**: Middle River, MD 21220
- **Service Area**: Baltimore, Hartford, and surrounding areas
- **Hours**: Open 24/7, 365 days a year
- **Established**: Since 2004

## Site Structure

### Main Pages
- **`index.html`** - Homepage with main content
- **`404.html`** - 404 error page
- **`hub/index.html`** - WPMU Dev Hub integration page (admin/management interface)

### Content Organization

The site follows WordPress's typical URL structure, organized into:

1. **Author Pages** (`author/` directory)
   - `alltypetowing19gmail-com/` - Author profile
   - `camsmithfreeforcharity-org/` - Author profile
   - `clarkemoyer/` - Author profile
   - `globaladmin/` - Global admin author

2. **Category Pages** (`category/` directory)
   - `uncategorized/` - Uncategorized posts

3. **Layout Components** (Divi theme structure)
   - `layout_category/header/` - Header layout
   - `layout_type/layout/` - Layout templates
   - `layout_type/row/` - Row layouts
   - `module_width/regular/` - Module width settings

4. **Scope Settings** (`scope/not_global/`) - Theme builder scope settings

### WordPress Assets

The site includes all WordPress assets:

- **`wp-content/`** - WordPress content directory
  - **`themes/Divi/`** - Divi theme files (CSS, JS, images)
  - **`plugins/`** - Installed WordPress plugins:
    - All-in-One WP Migration
    - Beehive Analytics
    - Forminator (form builder)
    - Hustle (popups/opt-ins)
    - Widget Google Reviews
    - WP Defender (security)
    - WP Smush Pro (image optimization)
    - WPMU Dev SEO (SmartCrawl)
    - And more...
  - **`uploads/`** - Media files (images from 2023, 2025)
  - **`et-cache/`** - Divi theme cache files

- **`wp-includes/`** - WordPress core files
  - Block styles and scripts
  - jQuery libraries
  - Core CSS/JS

## Site Features

### Navigation Menu
The site includes a main navigation with sections:
- **About** (`#about`)
- **Services** (`#services`)
- **Gallery** (`#gal`)
- **Reviews** (`#rev`)
- **Location** (`#map`)

### Services Offered
- Local and long-distance towing
- Cars, trucks, vans, motorcycles
- Exotic vehicle towing
- Roadside assistance
- 100% damage-free service record
- Insurance work
- Corporate/fleet contracts

### Social Media Links
- Facebook
- Google Business
- Yelp

## Technical Details

### Theme & Framework
- **Theme**: Divi v.4.27.5 (Elegant Themes)
- **WordPress Version**: 6.9
- **Export Method**: Static HTML export (likely using Simply Static or similar)

### Key Technologies
- HTML5
- CSS3 (extensive custom Divi styles)
- JavaScript (jQuery, Divi scripts)
- Responsive design (mobile, tablet, desktop breakpoints)
- Lazy loading for images (WP Smush Pro)

### Performance Features
- Cached CSS files
- Lazy-loaded images
- Minified JavaScript
- Optimized assets

## File Organization

```
FFC-EX-AllTypeTowing.com/
├── index.html (Homepage)
├── 404.html (Error page)
├── README.md (Project readme)
├── author/ (Author archive pages)
├── category/ (Category archive pages)
├── layout_*/ (Divi theme builder layouts)
├── hub/ (WPMU Dev Hub)
├── wp-content/ (WordPress assets)
│   ├── themes/Divi/ (Theme files)
│   ├── plugins/ (Plugin files)
│   ├── uploads/ (Media files)
│   └── et-cache/ (Divi cache)
└── wp-includes/ (WordPress core)
```

## Notes

1. **Static Export**: This is a static HTML export, meaning:
   - No WordPress database required
   - No PHP processing needed
   - Can be hosted on any static hosting (GitHub Pages, Netlify, etc.)
   - Forms and dynamic features may not work without backend

2. **Staging Site**: The site title indicates this is a "Staging Site", suggesting it may be a development/test version

3. **WPMU Dev Integration**: The site uses WPMU Dev plugins and Hub, indicating it's part of a managed WordPress hosting/development environment

4. **Free For Charity**: The Hub page references "Free For Charity" - this may be a charitable organization or hosting arrangement

## Recommendations

If you need to:
- **Edit content**: You'll need to edit the HTML files directly or restore to WordPress
- **Update images**: Replace files in `wp-content/uploads/`
- **Change styles**: Modify CSS in theme files or inline styles
- **Add functionality**: Consider restoring to WordPress or implementing with static site generators

## Contact Information (from site)

- **Phone**: (443) 740-0423
- **Address**: Middle River, MD 21220
- **Service**: 24/7/365 availability

