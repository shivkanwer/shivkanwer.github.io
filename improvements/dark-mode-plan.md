# Dark Mode Implementation Plan — The Teck Talk Blog

## Context

The blog uses Minimal Mistakes 4.24.0 with the "default" (light) skin. Minimal Mistakes has a built-in `_dark.scss` skin, but skins are compile-time only — the theme compiles ONE skin into `main.css` via SCSS variables. There is no built-in runtime toggle.

To support a user-togglable dark mode that also respects OS preference (`prefers-color-scheme`), we need to use **CSS custom properties** (runtime-switchable) layered on top of the SCSS-compiled base. This approach avoids modifying Minimal Mistakes core files and keeps upgrades safe.

---

## Architecture

```
User clicks toggle → JS sets data-theme="dark" on <html>
                    → JS saves preference to localStorage
                    → CSS custom properties switch all colors
                    → meta theme-color updates

Page load → JS checks localStorage
          → Falls back to prefers-color-scheme
          → Sets data-theme before first paint (no flash)
```

**Why CSS custom properties instead of two compiled CSS files?**
- Single CSS file (no extra HTTP request, no FOUC from stylesheet swap)
- Simpler build process (no changes to Jekyll build)
- Compatible with GitHub Pages (no custom plugins needed)
- Granular control over every color surface

---

## Files Overview

| File | Action | Purpose |
|------|--------|---------|
| `_sass/custom.scss` | Modify | Add CSS custom properties, dark mode values, update all color references |
| `_includes/head.html` | Modify | Add inline script for theme initialization (prevents flash) |
| `_includes/head/custom.html` | Modify | Update meta theme-color to use JS |
| `_includes/masthead.html` | Modify | Add dark mode toggle button |
| `index.html` | Modify | Update hero overlay color for dark mode awareness |
| `_config.yml` | No change | Keep "default" skin — we layer dark mode on top via CSS |

No new files needed — all changes go into existing files.

---

## Change 1: CSS Custom Properties Foundation

**File:** `_sass/custom.scss` — add at the very top, before existing design tokens

This defines the two color schemes using CSS custom properties on `<html>`. The existing SCSS variables (`$accent-color`, etc.) remain for SCSS-level usage; CSS custom properties handle runtime switching.

```scss
// === CSS Custom Properties for Theme Switching ===

// Light theme (default)
:root {
  // Page
  --color-bg: #fff;
  --color-bg-muted: #f8f9fa;
  --color-bg-surface: #fff;

  // Text
  --color-text: #3d3d3d;
  --color-text-primary: #333;
  --color-text-secondary: #555;
  --color-text-muted: #666;

  // Accent
  --color-accent: #007acc;
  --color-accent-hover: #00639e;
  --color-featured: #ffd93d;

  // Borders
  --color-border: #f2f3f3;

  // Shadows (opacity shifts in dark mode)
  --shadow-sm: 0 2px 8px rgba(0, 0, 0, 0.1);
  --shadow-md: 0 4px 16px rgba(0, 0, 0, 0.15);

  // Links — computed from theme: mix(#000, $info-color, 20%) where $info-color=#3b9cba
  --color-link: #2f7d95;
  --color-link-hover: #235d6f;

  // Code blocks
  --color-code-bg: #fafafa;

  // Footer
  --color-footer-bg: #f2f3f3;

  // Masthead
  --color-masthead-link: #6f777d;
  --color-masthead-link-hover: #3d4144;

  // Hero overlay
  --color-hero-overlay: #1f4e79;

  // Cards
  --color-card-bg: #fff;

  // Featured label
  --color-featured-text: #333;

  // Toggle icon
  --icon-theme-toggle: "\f186";  // moon icon (fa-moon)
}

// Dark theme
[data-theme="dark"] {
  // Page
  --color-bg: #1a1d23;
  --color-bg-muted: #252a34;
  --color-bg-surface: #252a34;

  // Text
  --color-text: #e0e0e0;
  --color-text-primary: #e0e0e0;
  --color-text-secondary: #b0b0b0;
  --color-text-muted: #999;

  // Accent (slightly brightened for dark backgrounds)
  --color-accent: #3a9fd8;
  --color-accent-hover: #5bb4e5;
  --color-featured: #ffd93d;

  // Borders
  --color-border: #3a3f48;

  // Shadows (stronger on dark to maintain depth perception)
  --shadow-sm: 0 2px 8px rgba(0, 0, 0, 0.3);
  --shadow-md: 0 4px 16px rgba(0, 0, 0, 0.4);

  // Links
  --color-link: #5bb4e5;
  --color-link-hover: #8ad0f5;

  // Code blocks (already dark-themed in Minimal Mistakes, just slightly adjust)
  --color-code-bg: #1e2229;

  // Footer
  --color-footer-bg: #14161a;

  // Masthead
  --color-masthead-link: #e0e0e0;
  --color-masthead-link-hover: #b0b0b0;

  // Hero overlay (deeper for dark mode)
  --color-hero-overlay: #0f2a43;

  // Cards
  --color-card-bg: #252a34;

  // Featured label (dark text on yellow still works)
  --color-featured-text: #1a1d23;

  // Toggle icon
  --icon-theme-toggle: "\f185";  // sun icon (fa-sun)
}

// Auto dark mode (respects OS preference when no explicit user choice)
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    // Same values as [data-theme="dark"] above
    --color-bg: #1a1d23;
    --color-bg-muted: #252a34;
    --color-bg-surface: #252a34;
    --color-text: #e0e0e0;
    --color-text-primary: #e0e0e0;
    --color-text-secondary: #b0b0b0;
    --color-text-muted: #999;
    --color-accent: #3a9fd8;
    --color-accent-hover: #5bb4e5;
    --color-featured: #ffd93d;
    --color-border: #3a3f48;
    --shadow-sm: 0 2px 8px rgba(0, 0, 0, 0.3);
    --shadow-md: 0 4px 16px rgba(0, 0, 0, 0.4);
    --color-link: #5bb4e5;
    --color-link-hover: #8ad0f5;
    --color-code-bg: #1e2229;
    --color-footer-bg: #14161a;
    --color-masthead-link: #e0e0e0;
    --color-masthead-link-hover: #b0b0b0;
    --color-hero-overlay: #0f2a43;
    --color-card-bg: #252a34;
    --color-featured-text: #1a1d23;
    --icon-theme-toggle: "\f185";
  }
}
```

**Design decisions:**
- Dark background `#1a1d23` is slightly darker than Minimal Mistakes' default dark skin (`#252a34`) for better contrast
- Accent blue brightened from `#007acc` to `#3a9fd8` to maintain WCAG AA contrast on dark backgrounds
- Shadows use higher opacity on dark (0.3/0.4 vs 0.1/0.15) to remain visible
- The `@media (prefers-color-scheme: dark)` block uses `:root:not([data-theme="light"])` so it only applies when the user hasn't explicitly chosen light mode

---

## Change 2: Override Minimal Mistakes Theme Colors

**File:** `_sass/custom.scss` — add after the CSS custom properties block

Minimal Mistakes uses SCSS variables compiled to static values. We override them at the CSS level using custom properties. These rules must come after the theme's compiled output (they do, since `custom.scss` is imported last in `main.scss`).

```scss
// === Theme Color Overrides (runtime-switchable) ===

html {
  background-color: var(--color-bg);
}

body {
  color: var(--color-text);
}

// Links
a {
  color: var(--color-link);

  &:hover {
    color: var(--color-link-hover);
  }
}

// Masthead — the theme does NOT set a background on .masthead (only on .greedy-nav
// inside it). We add one here to prevent transparent gaps in dark mode.
.masthead {
  background-color: var(--color-bg);
  border-bottom-color: var(--color-border);
}

.greedy-nav {
  background: var(--color-bg);

  a {
    color: var(--color-masthead-link);

    &:hover {
      color: var(--color-masthead-link-hover);
    }
  }

  .visible-links a:before {
    background: var(--color-accent);
  }

  button {
    color: var(--color-masthead-link);
  }
}

// Hidden nav links dropdown (mobile menu)
.greedy-nav .hidden-links {
  background: var(--color-bg-surface);
  box-shadow: var(--shadow-md);

  a {
    color: var(--color-text-primary);

    &:hover {
      color: var(--color-accent);
      background: var(--color-bg-muted);
    }
  }
}

// Footer
.page__footer {
  background-color: var(--color-footer-bg);
  color: var(--color-text-muted);

  a {
    color: var(--color-text-secondary);

    &:hover {
      color: var(--color-accent);
    }
  }
}

// Inline code only — fenced blocks (.highlighter-rouge) already use dark bg (#263238).
// A bare `code` selector would override fenced block backgrounds, so we target
// only inline code contexts.
p > code,
li > code,
td > code,
h2 > code,
h3 > code,
h4 > code {
  background: var(--color-code-bg);
  color: var(--color-text);
  border: 1px solid var(--color-border);
}

// Tables — theme applies background-color to `thead`, not `th`
table {
  color: var(--color-text);

  thead {
    background-color: var(--color-bg-muted);
    border-bottom-color: var(--color-border);
  }

  td {
    border-bottom-color: var(--color-border);
  }
}

// Notice boxes — override text AND background for dark mode.
// The theme compiles notice backgrounds using mix($background-color, $notice-color, 80%)
// where $background-color is #fff. In dark mode, these light-tinted backgrounds persist
// while text switches to light (#e0e0e0), creating light-on-light — unreadable.
// We must override backgrounds to darker tints.
.notice,
.notice--primary,
.notice--info,
.notice--warning,
.notice--success,
.notice--danger {
  color: var(--color-text);
}

[data-theme="dark"],
:root:not([data-theme="light"]) {
  .notice             { background-color: #2a2e36; }
  .notice--primary    { background-color: #2a3340; }
  .notice--info       { background-color: #1e3a4a; }
  .notice--warning    { background-color: #3a3520; }
  .notice--success    { background-color: #1e3a2a; }
  .notice--danger     { background-color: #3a1e1e; }
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    .notice             { background-color: #2a2e36; }
    .notice--primary    { background-color: #2a3340; }
    .notice--info       { background-color: #1e3a4a; }
    .notice--warning    { background-color: #3a3520; }
    .notice--success    { background-color: #1e3a2a; }
    .notice--danger     { background-color: #3a1e1e; }
  }
}

// Blockquotes
blockquote {
  border-left-color: var(--color-accent);
  color: var(--color-text-secondary);
}

// Page content
.page__content {
  h2 {
    border-bottom-color: var(--color-border);
  }
}

// Search
.search-content {
  background-color: var(--color-bg);

  .search-input {
    color: var(--color-text);
    background-color: var(--color-bg-surface);
    border-color: var(--color-border);
  }
}

// Pagination
.pagination a,
.pagination span {
  color: var(--color-text-muted);
  border-color: var(--color-border);
}

// TOC (if added later)
.toc {
  background-color: var(--color-bg-surface);
  border-color: var(--color-border);
}

// Breadcrumbs
.breadcrumbs {
  ol {
    color: var(--color-text-muted);

    a {
      color: var(--color-link);
    }
  }
}

// Sidebar (author profile pages)
.sidebar {
  .author__name {
    color: var(--color-text-primary);
  }

  .author__bio {
    color: var(--color-text-secondary);
  }

  .author__urls a {
    color: var(--color-text-secondary);

    &:hover {
      color: var(--color-accent);
    }
  }
}

// Archive pages
.taxonomy__section .archive__item-title a {
  color: var(--color-text-primary);

  &:hover {
    color: var(--color-accent);
  }
}

// Twitter/X share button — custom.scss overrides this to #000 with !important.
// On dark backgrounds (#1a1d23), a #000 button is nearly invisible (contrast ~1.15:1).
// Override to a visible dark gray in dark mode.
[data-theme="dark"] .btn--twitter {
  background-color: #333 !important;

  &:hover {
    background-color: #555 !important;
  }
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) .btn--twitter {
    background-color: #333 !important;

    &:hover {
      background-color: #555 !important;
    }
  }
}
```

---

## Change 3: Update Custom Component Colors

**File:** `_sass/custom.scss` — update all existing custom component rules

Replace SCSS variable references with CSS custom property references in the existing component styles. The SCSS variables at the top of the file remain defined (for any SCSS-level operations like `darken()`), but all runtime-visible color properties switch to `var(--...)`.

### Featured Label
```scss
/* Before */
.featured-label {
  background: $featured-color;
  color: $text-primary;
}

/* After */
.featured-label {
  background: var(--color-featured);
  color: var(--color-featured-text);
}
```

### Homepage Cards (`.splash .archive__item`)
```scss
/* Before */
.archive__item {
  background: $bg-surface;
  box-shadow: $shadow-sm;
  transition: transform $transition-fast, box-shadow $transition-fast;

  &:hover {
    box-shadow: $shadow-md;
  }

  .archive__item-title a {
    color: $text-primary;
    &:hover { color: $accent-color; }
  }

  .page__meta {
    color: $text-muted;
  }

  .archive__item-excerpt {
    color: $text-secondary;
  }
}

/* After */
.archive__item {
  background: var(--color-card-bg);
  box-shadow: var(--shadow-sm);
  transition: transform $transition-fast, box-shadow $transition-fast;

  &:hover {
    box-shadow: var(--shadow-md);
  }

  .archive__item-title a {
    color: var(--color-text-primary);
    &:hover { color: var(--color-accent); }
  }

  .page__meta {
    color: var(--color-text-muted);
  }

  .archive__item-excerpt {
    color: var(--color-text-secondary);
  }
}
```

### Category Section Background
```scss
/* Before */
.feature:last-child {
  background: $bg-muted;
  .archive__item { background: $bg-surface; }
  .archive__item-title { color: $text-primary; }
  .archive__item-excerpt p:first-child { color: $text-muted; }
}

/* After */
.feature:last-child {
  background: var(--color-bg-muted);
  .archive__item { background: var(--color-card-bg); }
  .archive__item-title { color: var(--color-text-primary); }
  .archive__item-excerpt p:first-child { color: var(--color-text-muted); }
}
```

### Heading Hierarchy
```scss
/* Before */
.wide-no-author .page__content {
  h2 { border-bottom: 1px solid $border-color; }
  h4 { color: $text-secondary; }
}

/* After */
.wide-no-author .page__content {
  h2 { border-bottom: 1px solid var(--color-border); }
  h4 { color: var(--color-text-secondary); }
}
```

### Author Link
```scss
/* Before */
.page__meta-author .author-link {
  &:hover { color: $accent-color; }
}

/* After */
.page__meta-author .author-link {
  &:hover { color: var(--color-accent); }
}
```

### Image Caption Shadows
```scss
/* Before */
.image-with-caption img {
  box-shadow: $shadow-sm;
}

/* After */
.image-with-caption img {
  box-shadow: var(--shadow-sm);
}
```

### Hero Author Section

The hero section sits on a dark overlay image, so its colors (white text, white borders) work in both light and dark modes. **No changes needed** for the hero author content — it's already designed for dark backgrounds.

One adjustment: the hero overlay color should deepen slightly in dark mode for visual cohesion.

In `index.html`, the overlay is defined as front matter (`overlay_color: "#1f4e79"`), which Minimal Mistakes renders as an **inline style** (`style="background-color: #1f4e79;"`) on the `.page__hero--overlay` element. Inline styles have specificity `(1,0,0)`, so `!important` is required to override them:

```scss
// Hero overlay dark mode adjustment
// IMPORTANT: !important is required because Minimal Mistakes applies
// overlay_color as an inline style from front matter
[data-theme="dark"] .page__hero--overlay {
  background-color: var(--color-hero-overlay) !important;
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) .page__hero--overlay {
    background-color: var(--color-hero-overlay) !important;
  }
}
```

---

## Change 4: Theme Toggle Button in Masthead

**File:** `_includes/masthead.html`

Add a toggle button next to the search button (line 22-25). It uses a FontAwesome icon that switches between moon (light mode) and sun (dark mode).

```html
<!-- Insert after line 25 (after search button {% endif %}) and before greedy-nav toggle (line 27) -->
<button class="theme-toggle" type="button" aria-label="Toggle dark mode" title="Toggle dark mode">
  <i class="fas fa-moon"></i>
</button>
```

**Updated masthead.html (lines 21–31):**
```html
        {% if site.search == true %}
        <button class="search__toggle" type="button">
          <span class="visually-hidden">{{ site.data.ui-text[site.locale].search_label | default: "Toggle search" }}</span>
          <i class="fas fa-search"></i>
        </button>
        {% endif %}
        <button class="theme-toggle" type="button" aria-label="Toggle dark mode" title="Toggle dark mode">
          <i class="fas fa-moon"></i>
        </button>
        <button class="greedy-nav__toggle hidden" type="button">
          <span class="visually-hidden">{{ site.data.ui-text[site.locale].menu_label | default: "Toggle menu" }}</span>
          <div class="navicon"></div>
        </button>
```

**Styling for the toggle button** (add to `_sass/custom.scss`):

```scss
// === Theme Toggle Button ===
.theme-toggle {
  display: inline-block;
  padding: 0.25em 0.5em;
  margin-left: 0.5em;
  background: none;
  border: none;
  cursor: pointer;
  color: var(--color-masthead-link);
  font-size: 1em;
  line-height: 1;
  transition: color $transition-fast, transform $transition-fast;
  vertical-align: middle;

  &:hover {
    color: var(--color-accent);
    transform: scale(1.1);
  }

  &:focus {
    outline: 2px solid var(--color-accent);
    outline-offset: 2px;
    border-radius: 4px;
  }

  // Swap icon based on theme
  [data-theme="dark"] & i:before {
    content: "\f185"; // fa-sun
  }

  @media (prefers-color-scheme: dark) {
    :root:not([data-theme="light"]) & i:before {
      content: "\f185"; // fa-sun
    }
  }
}
```

---

## Change 5: Theme Initialization Script (Prevent Flash)

**File:** `_includes/head.html`

Add an inline script immediately after the existing `no-js` script (line 12-14) and **before** the stylesheet link (line 17). This runs synchronously before first paint, preventing a flash of wrong theme. It also temporarily disables CSS transitions to prevent a color animation on page load (see Transition Animation section).

```html
<!-- Insert between line 14 and line 17 -->
<script>
  (function() {
    // Disable transitions during initial theme application to prevent flash animation
    document.documentElement.classList.add('no-transition');
    var stored = localStorage.getItem('theme');
    if (stored === 'dark' || stored === 'light') {
      document.documentElement.setAttribute('data-theme', stored);
    }
    // If no stored preference, don't set data-theme — let CSS @media rule handle
    // prefers-color-scheme. This way clearing localStorage returns to OS default.

    // Re-enable transitions after first paint
    requestAnimationFrame(function() {
      requestAnimationFrame(function() {
        document.documentElement.classList.remove('no-transition');
      });
    });
  })();
</script>
```

**Why inline and not an external file?** External scripts would load asynchronously and cause a flash of light theme before dark mode applies. This must run synchronously in `<head>`.

**Note:** This is the single, canonical initialization script. The Transition Animation section below provides the CSS for `no-transition` but does NOT contain a separate script — it uses this one.

---

## Change 6: Theme Toggle JavaScript

**File:** `_includes/head/custom.html`

Add the toggle logic. This goes in `head/custom.html` because it's included in `<head>` and is the designated place for custom snippets. It sets up the event listener after DOM load.

```html
<!-- Theme toggle logic -->
<script>
  document.addEventListener('DOMContentLoaded', function() {
    var toggle = document.querySelector('.theme-toggle');
    if (!toggle) return;

    // Update toggle icon based on current effective theme
    function updateIcon() {
      var icon = toggle.querySelector('i');
      if (!icon) return;
      var isDark = document.documentElement.getAttribute('data-theme') === 'dark' ||
        (!document.documentElement.getAttribute('data-theme') &&
         window.matchMedia('(prefers-color-scheme: dark)').matches);
      icon.className = isDark ? 'fas fa-sun' : 'fas fa-moon';
      toggle.setAttribute('aria-label', isDark ? 'Switch to light mode' : 'Switch to dark mode');
      toggle.setAttribute('title', isDark ? 'Switch to light mode' : 'Switch to dark mode');
    }

    // Update meta theme-color
    function updateMetaThemeColor() {
      var meta = document.querySelector('meta[name="theme-color"]');
      if (!meta) return;
      var isDark = document.documentElement.getAttribute('data-theme') === 'dark' ||
        (!document.documentElement.getAttribute('data-theme') &&
         window.matchMedia('(prefers-color-scheme: dark)').matches);
      meta.setAttribute('content', isDark ? '#1a1d23' : '#ffffff');
    }

    // Toggle theme
    toggle.addEventListener('click', function() {
      var current = document.documentElement.getAttribute('data-theme');
      var isDark;

      if (current === 'dark') {
        document.documentElement.setAttribute('data-theme', 'light');
        localStorage.setItem('theme', 'light');
        isDark = false;
      } else if (current === 'light') {
        document.documentElement.setAttribute('data-theme', 'dark');
        localStorage.setItem('theme', 'dark');
        isDark = true;
      } else {
        // No explicit theme — currently following OS preference
        // Toggle to opposite of OS preference
        var osPrefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
        var newTheme = osPrefersDark ? 'light' : 'dark';
        document.documentElement.setAttribute('data-theme', newTheme);
        localStorage.setItem('theme', newTheme);
        isDark = newTheme === 'dark';
      }

      updateIcon();
      updateMetaThemeColor();
    });

    // Listen for OS theme changes (user changes system preference)
    if (window.matchMedia) {
      window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', function() {
        // Only react if user hasn't set an explicit preference
        if (!localStorage.getItem('theme')) {
          updateIcon();
          updateMetaThemeColor();
        }
      });
    }

    // Initialize on load
    updateIcon();
    updateMetaThemeColor();
  });
</script>
```

---

## Change 7: Update Meta Theme-Color

**File:** `_includes/head/custom.html`

The existing static meta tag:
```html
<meta name="theme-color" content="#ffffff">
```

Keep it as-is. The JavaScript in Change 6 updates it dynamically. The static `#ffffff` serves as the fallback for no-JS environments.

No change needed to this line.

---

## Dark Mode Color Palette Reference

| Surface | Light | Dark | Notes |
|---------|-------|------|-------|
| Page background | `#fff` | `#1a1d23` | Slightly darker than MM default dark |
| Muted background | `#f8f9fa` | `#252a34` | Cards, sections |
| Card background | `#fff` | `#252a34` | Same as muted bg |
| Footer background | `#f2f3f3` | `#14161a` | Darkest surface |
| Primary text | `#333` | `#e0e0e0` | WCAG AA on both backgrounds |
| Secondary text | `#555` | `#b0b0b0` | |
| Muted text | `#666` | `#999` | |
| Accent | `#007acc` | `#3a9fd8` | Brightened for dark bg contrast |
| Accent hover | `#00639e` | `#5bb4e5` | |
| Featured label | `#ffd93d` on `#333` | `#ffd93d` on `#1a1d23` | Yellow works on both |
| Link | `#2f7d95` | `#5bb4e5` | Computed: mix(#000, #3b9cba, 20%) |
| Border | `#f2f3f3` | `#3a3f48` | $lighter-gray: mix(#fff, #7a8288, 90%) |
| Code bg (inline) | `#fafafa` | `#1e2229` | Fenced blocks unchanged |
| Hero overlay | `#1f4e79` | `#0f2a43` | Deeper blue in dark |
| Shadows | 0.1/0.15 alpha | 0.3/0.4 alpha | Stronger on dark |

---

## What Stays Unchanged

These elements work correctly in both modes without modification:

| Component | Why |
|-----------|-----|
| **Fenced code blocks** (`div.highlighter-rouge`) | Already use dark background (#263238) with light text — looks good in both modes |
| **Hero author section** (text, avatar, social icons) | Sits on dark overlay image — white text/borders work in both modes |
| **`.btn--primary` buttons** | Minimal Mistakes uses YIQ contrast at SCSS compile-time — button backgrounds (accent color) remain visible on both light and dark page backgrounds; text color is baked in at compile-time (not runtime-adaptive) |
| **Images in posts** | Content images don't need color changes |
| **FontAwesome icons** | Inherit text color via `currentColor` |

**Note on elements that DO need overrides (handled in Change 2):**
- **Social share buttons (Twitter/X):** Custom `.btn--twitter` uses `background-color: #000 !important` which is nearly invisible on dark backgrounds — overridden in Change 2
- **Notice boxes:** Theme compiles light-tinted backgrounds using `mix(#fff, $notice-color, 80%)`. In dark mode, text switches to light but backgrounds stay light-tinted — unreadable. Background overrides are in Change 2

---

## Image Handling in Dark Mode

Blog post images (diagrams, screenshots) may look jarring on dark backgrounds if they have white/light backgrounds. Add an optional subtle treatment:

```scss
// Soften images with white backgrounds in dark mode
[data-theme="dark"] .page__content img:not(.author__avatar img) {
  border-radius: $radius-sm;
  box-shadow: var(--shadow-sm);
}

// Optional: very slight brightness reduction for images in dark mode
// Uncomment if post images look too bright
// [data-theme="dark"] .page__content img {
//   filter: brightness(0.92);
// }
```

---

## Transition Animation

Add a smooth transition so the theme switch isn't jarring:

```scss
// Smooth theme transition (add to custom.scss)
html {
  transition: background-color 0.3s ease, color 0.3s ease;
}

body,
.masthead,
.page__footer,
.archive__item,
.greedy-nav,
.greedy-nav .hidden-links,
code,
thead {
  transition: background-color 0.3s ease, color 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
}

// Disable transition on initial page load to prevent flash animation.
// The `no-transition` class is added/removed by the initialization script in Change 5.
html.no-transition,
html.no-transition * {
  transition: none !important;
}
```

**Note:** No separate script is needed here. The initialization script in Change 5 already handles adding and removing the `no-transition` class.

---

## Accessibility Considerations

1. **Toggle button has `aria-label`** — dynamically updated to "Switch to light/dark mode"
2. **Focus indicator** — visible outline on toggle button when focused via keyboard
3. **Color contrast** — all dark mode text colors maintain WCAG AA contrast ratio (4.5:1 minimum):
   - `#e0e0e0` on `#1a1d23` = 12.8:1 (passes AAA)
   - `#b0b0b0` on `#1a1d23` = 7.8:1 (passes AAA)
   - `#999` on `#1a1d23` = 5.9:1 (passes AA)
   - `#3a9fd8` on `#1a1d23` = 5.7:1 (passes AA)
   - `#999` on `#252a34` (muted text on card bg) = 5.1:1 (passes AA, but tight margin — avoid for text smaller than 14pt bold)
4. **No motion issues** — transition is 0.3s and only affects color properties (not layout)
5. **Respects OS preference** — `prefers-color-scheme` honored by default; user toggle overrides

---

## Verification

### Automated Checks (run by Claude Code)

```bash
# 1. Jekyll build — confirms SCSS compiles
bundle exec jekyll build --strict_front_matter 2>&1 | tail -5

# 2. Verify CSS custom properties are defined
grep -c '\-\-color-bg:' _sass/custom.scss
# Expected: 3 (root, dark, prefers-color-scheme)

# 3. Verify toggle button exists in masthead
grep 'theme-toggle' _includes/masthead.html
# Expected: match found

# 4. Verify head initialization script exists
grep 'localStorage.*theme' _includes/head.html
# Expected: match found

# 5. Verify no hardcoded light-only colors remain in custom component rules
# (SCSS variable definitions at top are OK; references in component rules should use var())
grep -n '#007acc\|#333\|#555\|#666\|#fff\|#f8f9fa' _sass/custom.scss | grep -v '^[0-9]*:.*\$' | grep -v '\/\/' | grep -v 'rgba'
# Expected: only the SCSS variable definitions, not usage in component rules
```

### Manual Browser Testing

**Light mode (default):**
- [ ] Page loads in light mode — white background, dark text
- [ ] Toggle button shows moon icon in masthead
- [ ] All cards, headings, borders match current design
- [ ] No visual regressions from current site

**Dark mode (toggle):**
- [ ] Click toggle → page transitions smoothly to dark mode
- [ ] Toggle icon changes to sun
- [ ] Background becomes `#1a1d23`, text becomes `#e0e0e0`
- [ ] Cards have `#252a34` background with visible shadows
- [ ] Category section has slightly different dark shade
- [ ] Featured label: yellow badge with dark text still readable
- [ ] Links are lighter blue (`#5bb4e5`), visible against dark background
- [ ] Masthead and footer darken appropriately
- [ ] Code blocks (fenced) look unchanged — already dark
- [ ] Inline code (`backtick`) has dark background with light text
- [ ] Hero section looks good — overlay deepens, text stays white
- [ ] Tables have dark headers and visible borders
- [ ] Notice boxes have dark-tinted backgrounds with readable light text (not light-on-light)
- [ ] Twitter/X share button is visible (dark gray, not black) on dark page background
- [ ] Blockquote border uses accent color

**Persistence:**
- [ ] Refresh page → dark mode persists (localStorage)
- [ ] Open new tab → dark mode persists
- [ ] Toggle back to light → light mode persists on refresh

**OS preference:**
- [ ] Clear localStorage → page follows OS dark/light preference
- [ ] Set OS to dark → page auto-darkens (no toggle needed)
- [ ] Toggle overrides OS preference
- [ ] Change OS preference while page open → updates if no stored preference

**Responsive:**
- [ ] Toggle button visible and tappable on mobile
- [ ] Dark mode works at all breakpoints (480px, 768px, 1024px, desktop)
- [ ] Mobile nav dropdown darkens properly

**No flash:**
- [ ] Set dark mode → hard refresh → no flash of light theme
- [ ] Disable JS → page renders in light mode (graceful fallback)

**Accessibility:**
- [ ] Tab to toggle button → focus ring visible
- [ ] Screen reader announces "Switch to dark/light mode"
- [ ] All text passes WCAG AA contrast check in dark mode
