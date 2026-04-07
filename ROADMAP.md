# Swift.org Website Improvement Roadmap

This document captures findings from a comprehensive audit of the Swift.org website codebase, organized as an actionable improvement roadmap. Items are grouped by priority tier, with each item describing the issue, affected files, and recommended action.

---

## Critical

Issues that cause runtime errors or represent fundamental architectural problems.

### 1. Missing `copyToClipboard()` JavaScript Function

**Issue:** Nine component templates call `copyToClipboard()` via inline `onclick` handlers, but the function is never defined in any JavaScript file. This causes runtime errors when users click copy buttons on SDK installation pages.

**Affected files:**
- `_includes/new-includes/components/android-sdk.html` (line 10)
- `_includes/new-includes/components/android-sdk-dev.html` (lines 12, 33)
- `_includes/new-includes/components/static-linux-sdk.html` (line 10)
- `_includes/new-includes/components/static-linux-sdk-dev.html` (lines 13, 34)
- `_includes/new-includes/components/wasm-sdk.html` (line 10)
- `_includes/new-includes/components/wasm-sdk-dev.html` (lines 5, 26)

**Action:** Define `copyToClipboard(button, text)` in `assets/javascripts/new-javascripts/application.js` using the Clipboard API with a fallback. Migrate the inline `onclick` handlers to event listeners attached via JavaScript to improve CSP compatibility.

### 2. Parallel Old/New Architecture

**Issue:** The codebase maintains two complete parallel architectures (old and new) across layouts, includes, stylesheets, JavaScript, and data files. This doubles the maintenance surface, creates confusion about which version is authoritative, and leads to divergent behavior.

**Affected directories:**
- `_layouts/` vs `_layouts/new-layouts/` (91 lines vs 207 lines in `base.html`)
- `_includes/` vs `_includes/new-includes/`
- `_data/` vs `_data/new-data/`
- `assets/javascripts/` vs `assets/javascripts/new-javascripts/`
- `assets/stylesheets/` vs `assets/stylesheets/new-stylesheets/` (37 imports vs 8 imports in `application.scss`)

**Metrics:**
- 113 files use the `page` layout (old)
- 162 files use `new-layouts/post` layout
- 9 files use `new-layouts/base` layout
- 13 files use `new-layouts/install-linux-version` layout

**Action:** Audit which pages still use old layouts and migrate them to the new architecture. Once all pages use the new layouts, remove the old directories entirely. This is a phased effort that should be tracked with a migration checklist.

---

## High

Issues affecting accessibility, CI reliability, and content correctness.

### 3. Accessibility: Missing Image Alt Attributes

**Issue:** At least 21 component templates render `<img>` tags without `alt` attributes, violating WCAG 2.1 Level A compliance.

**Affected files (partial list):**
- `_includes/new-includes/components/carousel.html`
- `_includes/new-includes/components/code-box.html`
- `_includes/new-includes/components/code-text-row.html`
- `_includes/new-includes/components/get-started-hero.html`
- `_includes/new-includes/components/headline-section.html`
- `_includes/new-includes/components/link-columns.html`
- `_includes/new-includes/components/linux-os-selection.html`
- `_includes/new-includes/components/linux-releases.html`
- `_includes/new-includes/components/os-selection.html`
- `_includes/new-includes/components/overlapping-containers.html`
- `_includes/new-includes/components/tab-nav.html`
- `_includes/new-includes/components/android-sdk.html`
- `_includes/new-includes/components/android-sdk-dev.html`
- `_includes/new-includes/components/static-linux-sdk.html`
- `_includes/new-includes/components/static-linux-sdk-dev.html`
- `_includes/new-includes/components/wasm-sdk.html`
- `_includes/new-includes/components/wasm-sdk-dev.html`

**Action:** Add `alt` attributes to all `<img>` tags. Use descriptive text from include parameters (e.g., `{{ include.alt | default: "" }}`). For decorative images, use `alt=""`. Also remove redundant ARIA roles (`role="main"` on `<main>`, `role="navigation"` on `<nav>`).

### 4. No HTML Validation, Link Checking, or Accessibility Testing in CI

**Issue:** The CI pipeline (`.github/workflows/ci.yaml`) has only three jobs: `soundness` (license headers), `build-site` (build with `--strict_front_matter`), and `test-openapi`. There is no validation of the generated HTML, no broken link detection, no accessibility checks, and no artifact storage.

**Affected files:**
- `.github/workflows/ci.yaml`

**Action:** Add CI jobs for:
- HTML validation (html-proofer or similar)
- Broken link detection (internal and external)
- Accessibility checks (axe-core or Pa11y)
- Save build artifacts for inspection on failure

### 5. No Automated Dependency Scanning

**Issue:** No Dependabot configuration exists. Dependencies are updated manually (evidenced by "Bundle update" commits). No security scanning (Snyk, SBOM generation, or secret scanning) is configured.

**Affected files:**
- `.github/` (missing `dependabot.yml`)
- `Gemfile`, `Gemfile.lock`, `package.json`

**Action:** Add `.github/dependabot.yml` covering both `bundler` and `npm` ecosystems. Consider adding GitHub's code scanning for secret detection.

### 6. Blog Post Date Format Inconsistencies

**Issue:** Four blog post filenames use non-zero-padded months or days, deviating from the Jekyll `YYYY-MM-DD` convention. While Jekyll handles this, it creates inconsistency and may cause sorting issues.

**Affected files:**
- `_posts/2016-1-05-swift-2.2-release-process.md` (should be `2016-01-05`)
- `_posts/2017-2-16-swift-4.0-release-process.md` (should be `2017-02-16`)
- `_posts/2017-3-27-swift-3.1-released.md` (should be `2017-03-27`)
- `_posts/2016-12-9-swift-3.1-release-process.md` (should be `2016-12-09`)

**Action:** Rename the files to use zero-padded dates. Add `redirect_from` in front matter to preserve old URLs if they were ever linked externally.

### 7. FIXME/TODO Comments in Production Code

**Issue:** Three FIXME/TODO comments indicate incomplete implementations shipped to production.

**Affected files:**
- `_includes/new-includes/components/code-image-column.html` (line 25): `<!-- FIXME: Include body & link -->`
- `_includes/new-includes/components/full-width-text-image-column.html` (line 15): `<!-- FIXME: Move body & link below image -->`
- `assets/stylesheets/new-stylesheets/_core.scss` (line 30): `// TODO: Remove this and move the rule in body`

**Action:** Resolve each TODO/FIXME by implementing the described change or, if no longer relevant, removing the comment. Track in issues if the work is non-trivial.

### 8. Inline `onclick` Handlers

**Issue:** Nine instances of inline `onclick="copyToClipboard(...)"` in templates. Beyond the missing function (Critical #1), inline handlers violate Content Security Policy best practices and are a deprecated pattern.

**Affected files:** Same as Critical #1.

**Action:** After implementing `copyToClipboard()`, refactor these to use `data-` attributes and attach event listeners from JavaScript. Example: `<button data-copy-text="{{ command | escape }}">` with a delegated click handler.

---

## Medium

Code quality, maintainability, and consistency issues.

### 9. Duplicate JavaScript Functions

**Issue:** `toggleClass()` is defined in both `assets/javascripts/application.js` (lines 26-37, IIFE pattern) and `assets/javascripts/new-javascripts/application.js` (lines 1-15, module pattern) with different implementations. Copy button handling is also duplicated with divergent behavior (old: text buttons; new: SVG icons).

**Affected files:**
- `assets/javascripts/application.js` (111 lines)
- `assets/javascripts/new-javascripts/application.js`

**Action:** Consolidate as part of the old/new architecture migration (Critical #2). Until then, ensure both implementations are functionally equivalent.

### 10. Hardcoded CSS Values

**Issue:** `assets/stylesheets/new-stylesheets/_core.scss` uses the hardcoded color `#051416` (line 12) instead of a CSS variable, while other properties in the same file use `var(--...)` notation (line 62). No centralized breakpoint system is documented despite `--navigation-mobile-breakpoint` and `--content-mobile-breakpoint` both being set to `68rem`.

**Affected files:**
- `assets/stylesheets/new-stylesheets/_core.scss` (lines 12, 62)
- `assets/stylesheets/core/_vars.scss`

**Action:** Replace hardcoded values with CSS custom properties. Document the breakpoint system.

### 11. Plugin Robustness Issues

**Issue:** Several Jekyll plugins have robustness concerns:
- `liquid-in-yaml.rb` (lines 21-24): A `while` loop re-renders Liquid templates until stable, but has no iteration limit, risking infinite loops on malformed input.
- `liquid-in-yaml.rb` (line 38): Uses generic `rescue => e` that catches all exceptions silently.
- `packages.rb` (lines 11, 27): Interpolates `category['slug']` directly into Liquid template strings without sanitization.
- `convert-header.rb` (lines 18-20): Embeds an SVG with 1800+ character `d` paths directly in Ruby source code.

**Affected files:**
- `_plugins/liquid-in-yaml.rb`
- `_plugins/packages.rb`
- `_plugins/convert-header.rb`

**Action:**
- Add a maximum iteration count (e.g., 10) to the `while` loop in `liquid-in-yaml.rb` and raise a descriptive error if exceeded.
- Narrow `rescue` to specific exception types (e.g., `Liquid::SyntaxError`).
- Sanitize `category['slug']` before interpolation in `packages.rb`.
- Extract the SVG from `convert-header.rb` into a separate file and load it at build time.

### 12. Missing Linting Tools

**Issue:** No JavaScript, CSS, Markdown, or YAML linting is configured. Prettier is present but only covers `new-*` directories.

**Affected files:**
- `package.json` (only has Prettier)
- `.prettierrc`, `.prettierignore`

**Action:** Add and configure:
- ESLint for JavaScript
- Stylelint for SCSS/CSS
- markdownlint for Markdown content
- yamllint for YAML data files
- rubocop for Ruby plugins

Add corresponding CI jobs.

### 13. Inconsistent YAML File Extensions

**Issue:** Most data files use `.yml` but `_data/documentation.yaml` uses `.yaml`. This creates inconsistency and can cause confusion.

**Affected files:**
- `_data/documentation.yaml` (5.9KB)
- All other data files use `.yml`

**Action:** Standardize on `.yml` (the existing majority convention). Rename `documentation.yaml` to `documentation.yml` and update any references.

### 14. Duplicate `get-started/` and `getting-started/` Directories

**Issue:** `get-started/` exists as a redirect to `/getting-started/`, but both directories are maintained with different subdirectory structures.

**Affected directories:**
- `get-started/`
- `getting-started/`

**Action:** Verify that `get-started/` only contains redirects. If so, consider consolidating. If `get-started/` URLs are linked externally, keep the redirects but document the relationship.

### 15. Large GIF Files in Assets

**Issue:** Three GIF files in `assets/images/gsoc-25/` are significantly large for web delivery.

**Affected files:**
- `assets/images/gsoc-25/output.gif` (4.3MB)
- `assets/images/gsoc-25/full.gif` (3.1MB)
- `assets/images/gsoc-25/brief.gif` (1.5MB)

**Action:** Convert GIFs to MP4 or WebM video, which typically achieves 80-90% size reduction. Use `<video>` tags with `autoplay muted loop playsinline` for GIF-like behavior. Alternatively, use animated WebP.

### 16. Docker Service Duplication

**Issue:** `docker-compose.yaml` defines `build` and `test` services (lines 34-37 vs 40-43) with identical commands. The test service provides no additional testing beyond a standard build.

**Affected files:**
- `docker-compose.yaml`

**Action:** Extend the `test` service to run actual validation (HTML-proofer, linting) after building. This makes the test service meaningful.

### 17. Blog Posts Missing Author Field

**Issue:** Three of the earliest blog posts lack an `author` field in their front matter.

**Affected files:**
- `_posts/2015-12-03-welcome.md`
- `_posts/2015-12-03-swift-3-api-design.md`
- `_posts/2015-12-03-swift-linux-port.md`

**Action:** Add appropriate author fields. Check `_data/authors.yml` for the correct author key to use.

---

## Low

Nice-to-have improvements for developer experience, SEO, and long-term maintenance.

### 18. Inline Styles in Templates

**Issue:** Some templates use inline `style` attributes instead of CSS classes.

**Affected files:**
- `_includes/new-includes/components/icon.html` (line 5): `style="width:{{ include.size | default: '24px' }}; height:{{ include.size | default: '24px' }}"`

**Action:** Replace inline styles with CSS classes using custom properties for dynamic sizing (e.g., `style="--icon-size: {{ include.size | default: '24px' }}"` with CSS `width: var(--icon-size)`).

### 19. Missing Structured Data (JSON-LD)

**Issue:** The site has Open Graph and Twitter Card meta tags but lacks JSON-LD structured data for blog posts, authors, and breadcrumbs. This limits search engine rich result eligibility.

**Affected files:**
- `_layouts/base.html`
- `_layouts/new-layouts/base.html`
- `_layouts/new-layouts/post.html`

**Action:** Add JSON-LD `Article` schema to blog post layouts and `BreadcrumbList` schema to navigation. Consider using the `jekyll-seo-tag` plugin.

### 20. No `.editorconfig`

**Issue:** No `.editorconfig` file exists to enforce consistent formatting (indentation, line endings, trailing whitespace) across editors and contributors.

**Action:** Create `.editorconfig` at the project root with settings matching the existing codebase conventions (2-space indent for HTML/SCSS/JS/YAML/Ruby, UTF-8, LF line endings).

### 21. No Performance Monitoring (Lighthouse CI)

**Issue:** No automated performance monitoring exists. There is no Lighthouse CI integration or Web Vitals tracking.

**Action:** Add Lighthouse CI to the GitHub Actions workflow to run on PRs. Set performance budgets for key pages (homepage, blog index, getting-started).

### 22. Missing Troubleshooting Section in README

**Issue:** The README provides setup instructions but no troubleshooting guidance for common issues (Ruby version conflicts, Bundler errors, Docker problems).

**Affected files:**
- `README.md`

**Action:** Add a "Troubleshooting" section covering common setup issues. Include guidance for macOS, Linux, and Docker environments.

### 23. Timezone Configuration

**Issue:** `_config.yml` (line 4) sets `timezone: America/Lower_Princes`, which appears to be an unusual timezone choice. Verify this is intentional.

**Affected files:**
- `_config.yml`

**Action:** Verify the timezone is correct. If it was set arbitrarily, consider using `UTC` or a more common timezone.

### 24. Content Archival Strategy

**Issue:** The repository contains 8 years of GSoC directories (`gsoc2019` through `gsoc2026`), 4 migration guide directories (Swift 3-5), and 3 information architecture proposal documents in `_info-architecture/`. No archival strategy exists for aging content.

**Action:** Consider grouping historical content under an `/archive/` path or adding "archived" banners to old pages. Create a policy for when content should be archived.

### 25. Missing Jekyll Plugins

**Issue:** The site manually implements functionality that established Jekyll plugins handle automatically.

**Missing plugins:**
- `jekyll-seo-tag` (automatic meta tag generation)
- `jekyll-sitemap` (currently manually maintained)
- `jekyll-minifier` (HTML/CSS/JS optimization)

**Action:** Evaluate adopting these plugins to reduce custom code and improve maintainability.

---

## Summary

| Priority | Count | Categories |
|----------|-------|------------|
| Critical | 2 | Runtime bug, Architecture |
| High | 6 | Accessibility, CI/CD, Content, Security |
| Medium | 9 | Code quality, Plugins, Tooling, Content, Performance |
| Low | 8 | DX, SEO, Configuration, Content |
| **Total** | **25** | |

### Key Metrics

- **162** blog posts across 5 categories
- **208** image files in assets (~51MB total)
- **353** data files across `_data/`
- **5** duplicate directory pairs (old/new architecture)
- **21+** components missing image alt attributes
- **9** instances of inline `onclick` calling an undefined function
- **3** FIXME/TODO comments in production
- **4** blog posts with malformed date filenames
- **~8.9MB** in oversized GIF files
