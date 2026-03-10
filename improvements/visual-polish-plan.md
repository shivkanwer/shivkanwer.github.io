# Visual Polish Plan — The Teck Talk Blog

## Context

The blog uses Minimal Mistakes 4.24.0 with extensive custom CSS in `_sass/custom.scss` and a custom homepage in `index.html`. While functionally solid and responsive, the styling has accumulated inconsistencies: hardcoded color/shadow/radius values scattered throughout, emoji labels that undermine the professional tone, a heading hierarchy that's too subtle to distinguish levels, and duplicated CSS across media queries. This plan addresses all of these to produce a cleaner, more consistent, and more maintainable design system.

---

## Files Modified

| File | Changes |
|------|---------|
| `_sass/custom.scss` | Add design tokens, replace hardcoded values, refactor hero section, fix headings, remove `!important`, add utility classes |
| `index.html` | Remove emojis, remove inline styles, add semantic CSS classes |

---

## Change 1: Define SCSS Design Tokens (variables)

**File:** `_sass/custom.scss` (added at top of file)

Add a variables block to centralize all design values. Since the theme uses `!default`, our variables take precedence.

```scss
// === Design Tokens ===

// Colors
$accent-color: #007acc;
$featured-color: #ffd93d;
$text-primary: #333;
$text-secondary: #555;
$text-muted: #666;
$bg-surface: #fff;
$bg-muted: #f8f9fa;

// Shadows
$shadow-sm: 0 2px 8px rgba(0, 0, 0, 0.1);
$shadow-md: 0 4px 16px rgba(0, 0, 0, 0.15);
$shadow-hero: 0 3px 10px rgba(0, 0, 0, 0.2);

// Border Radius
$radius-sm: 8px;
$radius-md: 10px;
$radius-lg: 12px;

// Spacing
$gap-sm: 1em;
$gap-md: 1.5em;
$gap-lg: 2em;

// Transitions
$transition-fast: 0.2s ease;
$transition-normal: 0.3s ease;
```

Then replace every hardcoded instance throughout the file:

| Hardcoded Value | Variable | Occurrences |
|-----------------|----------|-------------|
| `#007acc` | `$accent-color` | 2 |
| `#ffd93d` | `$featured-color` | 2 |
| `#333` | `$text-primary` | 3 |
| `#555` | `$text-secondary` | 1 |
| `#666` | `$text-muted` | 2 |
| `#fff` (in `.splash`) | `$bg-surface` | 2 |
| `#f8f9fa` | `$bg-muted` | 1 |
| `0 2px 8px rgba(0,0,0,0.1)` | `$shadow-sm` | 2 |
| `0 4px 16px rgba(0,0,0,0.15)` | `$shadow-md` | 1 |
| `0 3px 10px rgba(0,0,0,0.2)` | `$shadow-hero` | 1 |
| `border-radius: 8px` | `$radius-sm` | 3 |
| `border-radius: 10px` | `$radius-md` | 1 |
| `border-radius: 12px` | `$radius-lg` | 1 |
| `0.2s ease` | `$transition-fast` | 3 |
| `0.3s ease` | `$transition-normal` | 2 |
| `gap: 1.5em` | `$gap-md` | 2 |
| `gap: 2em` | `$gap-lg` | 1 |

---

## Change 2: Remove Emoji Labels from Homepage

**File:** `index.html`

| Line | Current | Replacement |
|------|---------|------------|
| 52 | `🌟 Featured Articles` | `Featured Articles` |
| 70 | `⭐ Featured` (badge text) | `Featured` |
| 86 | `📰 Recent Articles` | `Recent Articles` |
| 120 | `🔍 Explore by Topic` | `Explore by Topic` |
| 125 | `🤖 AI & Machine Learning` | `AI & Machine Learning` |
| 135 | `☸️ Kubernetes & Cloud Native` | `Kubernetes & Cloud Native` |
| 145 | `🏗️ Cloud Architecture` | `Cloud Architecture` |

The `.featured-label` CSS badge styling is kept — only the emoji character is removed.

---

## Change 3: Replace Inline Styles with CSS Classes

**File:** `index.html` — remove inline `style` attributes

| Line | Current inline style | Replacement |
|------|---------------------|-------------|
| 52 | `style="text-align: center; margin: 2em 0 1em 0;"` | Remove (handled by new `.splash h2` margin rule in CSS — see below) |
| 85 | `style="margin-top: 3em;"` | Add class `feature--recent` |
| 86 | `style="text-align: center; margin: 2em 0 1em 0;"` | Remove (handled by new `.splash h2` margin rule in CSS — see below) |
| 113 | `style="text-align: center; margin-top: 2em;"` | Add class `section__cta` |
| 119 | `style="margin-top: 3em; background: #f8f9fa; padding: 2em; border-radius: 8px;"` | Remove (handled by `.feature:last-child` in CSS — **note:** border-radius changes from 8px to 12px and vertical padding from 2em to 1.5em; this is intentional standardization) |
| 120 | `style="text-align: center; margin-bottom: 1em;"` | Remove (handled by `.splash h2`) |
| 124, 134, 144 | `style="text-align: center;"` | Remove (handled by `.feature:last-child .archive__item` in CSS) |

**File:** `_sass/custom.scss` — update `.splash h2` rule and add new utility classes

The existing `.splash h2, h3` rule (line 81) only sets `text-align: center`. It must also absorb the margin that was previously inline on the section headings, otherwise the Featured and Recent headings lose their spacing.

```scss
/* Center align all section headings */
.splash h2, h3 {
  text-align: center;
  margin: 2em 0 1em 0;
}

.feature--recent {
  margin-top: 3em;
}

.section__cta {
  text-align: center;
  margin-top: $gap-lg;
}
```

---

## Change 4: Fix Heading Hierarchy

**File:** `_sass/custom.scss` — replace `.wide-no-author .page__content` block (lines 525–560)

**Problem:** H2 (1.25em) and H3 (1.05em) differ by only 0.2em — too subtle for readers scanning long posts.

**Current:**
```scss
h2 { font-size: 1.25em !important; }
h3 { font-size: 1.05em !important; }
h4 { font-size: 0.9em !important; }
```

**Proposed:**
```scss
.wide-no-author .page__content {
  h2 {
    font-size: 1.4em;              // bumped from 1.25em
    margin: 1.75em 0 0.75em 0;
    font-weight: 700;
    border-bottom: 1px solid $border-color;  // visual section separator (uses Minimal Mistakes theme variable)
    padding-bottom: 0.3em;
  }

  h3 {
    font-size: 1.15em;             // 0.25em gap from h2 (was 0.2em)
    margin: 1.5em 0 0.5em 0;
    font-weight: 600;
  }

  h4 {
    font-size: 1.0em;              // 0.15em gap from h3
    margin: 1.25em 0 0.5em 0;
    font-weight: 600;
    color: $text-secondary;        // subtle color distinction
  }

  h5 {
    font-size: 0.9em;
    margin: 1em 0 0.5em 0;
    font-weight: 700;
  }

  h6 {
    font-size: 0.85em;
    margin: 1em 0 0.5em 0;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  blockquote {
    margin: 0;
  }
}
```

**Key improvements:**
- H2→H3 gap: 0.25em (1.4→1.15) instead of 0.2em (1.25→1.05)
- H2 gets a subtle bottom border for clear section breaks in long posts
- H4 uses color distinction (`$text-secondary`) rather than just size
- All `!important` flags removed — scoping to `.wide-no-author .page__content` provides sufficient specificity

---

## Change 5: Standardize Border Radius

**File:** `_sass/custom.scss`

| Element | Current | Proposed | Variable |
|---------|---------|----------|----------|
| `.image-with-caption img` | `8px` | `8px` | `$radius-sm` |
| `.featured-label` | `8px` | `8px` | `$radius-sm` |
| `.splash .archive__item` | `8px` | `8px` | `$radius-sm` |
| `.hero-author` | `10px` | `10px` | `$radius-md` |
| `.feature:last-child` | `12px` | `12px` | `$radius-lg` |

No visual change — this is a maintainability improvement only, replacing magic numbers with named variables.

---

## Change 6: Refactor Hero Author Section

**File:** `_sass/custom.scss` (lines 287–511)

**Problem:** 225 lines with 19 nested media queries scattered across 7 child elements (container + 6 children, each with 2–4 breakpoints). Nearly identical patterns repeated inside each child.

**Strategy:** Consolidate into 3 top-level media queries that handle all children at once.

**Before (abbreviated pattern):**
```scss
.hero-author {
  // base styles + 3 media queries for container
  .author__avatar img {
    // base + 3 nested media queries
  }
  .author__content {
    // base + 2 nested media queries
  }
  .author__name {
    // base + 3 nested media queries
  }
  .author__bio {
    // base + 3 nested media queries
  }
  .author__urls {
    // base + 2 nested media queries
  }
  .author__urls a {
    // base + 3 nested media queries
  }
}
// Total: ~225 lines, 19 media query blocks
```

**After:**
```scss
.hero-author {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: $gap-md;
  margin-top: 1.25em;
  padding: $gap-md $gap-lg;
  background: rgba(255, 255, 255, 0.1);
  border-radius: $radius-md;
  backdrop-filter: blur(10px);

  // --- Base child styles (desktop) ---
  .author__avatar {
    display: block;
    width: auto;
    height: auto;
    flex-shrink: 0;

    img {
      width: 120px;
      height: 120px;
      border-radius: 50%;
      border: 3px solid rgba(255, 255, 255, 0.8);
      box-shadow: $shadow-hero;
      padding: 0;
      object-fit: cover;
    }
  }

  .author__content {
    display: block;
    width: auto;
    padding: 0;
    color: white;
  }

  .author__name {
    margin: 0 0 0.35em 0;
    font-weight: 600;
    color: white;
    line-height: 1.2;
    font-size: 1.5em;
  }

  .author__bio {
    margin: 0 0 0.75em 0;
    line-height: 1.3;
    color: rgba(255, 255, 255, 0.9);
    font-size: 1em;
  }

  .author__urls {
    display: flex !important;      // KEEP: overrides theme's display:none
    position: static !important;   // KEEP: overrides theme positioning
    margin: 0 !important;
    padding: 0 !important;
    border: none !important;
    background: transparent !important;
    box-shadow: none !important;
    list-style-type: none !important;
    gap: 0.8em;
    justify-content: flex-start;
    flex-wrap: wrap;

    &:before, &:after { display: none !important; }

    a {
      color: rgba(255, 255, 255, 0.8) !important;
      transition: all $transition-normal;
      text-decoration: none !important;
      border: none !important;
      padding: 0 !important;
      margin: 0 !important;
      background: transparent !important;
      display: inline-block !important;
      font-size: 1.4em;

      &:hover {
        color: $featured-color !important;
        transform: translateY(-2px);
      }

      i {
        display: inline-block !important;
        opacity: 1 !important;
        visibility: visible !important;
      }
    }
  }

  // --- Tablet (769px–1024px) ---
  @media (min-width: 769px) and (max-width: 1024px) {
    gap: 1.25em;
    padding: 1.25em $gap-md;
    .author__avatar img { width: 100px; height: 100px; border-width: 2px; }
    .author__name { font-size: 1.35em; }
    .author__bio { font-size: 0.95em; }
    .author__urls a { font-size: 1.3em; }
  }

  // --- Mobile (≤768px) ---
  @media (max-width: 768px) {
    flex-direction: column;
    gap: $gap-sm;
    text-align: center;
    padding: 1.25em $gap-sm;
    margin-top: 1em;
    .author__avatar img { width: 85px; height: 85px; border-width: 2px; }
    .author__content { text-align: center; max-width: 100%; }
    .author__name { font-size: 1.25em; margin-bottom: 0.5em; }
    .author__bio { font-size: 0.9em; line-height: 1.4; margin-bottom: 1em; }
    .author__urls { gap: $gap-sm; justify-content: center; }
    .author__urls a { font-size: 1.25em; }
  }

  // --- Small mobile (≤480px) ---
  @media (max-width: 480px) {
    gap: 0.75em;
    padding: $gap-sm;
    margin-top: 0.75em;
    .author__avatar img { width: 70px; height: 70px; }
    .author__content { max-width: 280px; }
    .author__name { font-size: 1.1em; margin-bottom: 0.4em; }
    .author__bio { font-size: 0.85em; line-height: 1.3; margin-bottom: 0.75em; }
    .author__urls { gap: 0.8em; }
    .author__urls a { font-size: 1.15em; }
  }
}
```

**Impact:** ~225 lines → ~130 lines. Reduces 19 media query blocks down to 3.

**Note on `!important`:** The `author__urls` block retains `!important` flags because they override Minimal Mistakes' default `display: none` on `.author__urls` at smaller breakpoints. These are necessary for the hero section to work correctly. All other `!important` flags in the hero section are removed.

---

## Change 7: Eliminate Unnecessary `!important` Flags

**File:** `_sass/custom.scss`

| Location | Property | Action | Reason |
|----------|----------|--------|--------|
| `.splash h2, h3` | `text-align: center` | Remove `!important` | `.splash` parent provides sufficient specificity |
| `.splash .feature:last-child h2` | `margin` | Remove `!important` | Increase selector specificity instead |
| `.hero-author .author__name` | `margin, font-size, color` | Remove `!important` | Already scoped under `.hero-author` |
| `.hero-author .author__bio` | `margin, color` | Remove `!important` | Already scoped under `.hero-author` |
| `.hero-author .author__avatar img` | `border` | Remove `!important` | Scoped selector is specific enough |
| `.hero-author .author__avatar img` | `padding: 0` | **Keep** | Overrides theme default padding |
| `.hero-author .author__urls` | All properties | **Keep** | Required to override theme's `display: none` |
| `.hero-author .author__urls a` | All properties | **Keep** | Required to override theme link styles |
| `body.layout--splash .page__hero--overlay #page-title` | `display: none` | **Keep** | High specificity needed for theme override |
| `.page__hero--overlay` | `padding` | Remove `!important` | Change selector to `body.layout--splash .page__hero--overlay` for specificity (see code below) |
| `.wide-no-author .page__content h2–h6` | `font-size, font-weight` | Remove all `!important` | `.wide-no-author .page__content` is specific enough |
| `.wide-no-author .page__content blockquote` | `margin` | Remove `!important` | Same reason |

**`.page__hero--overlay` selector change** (lines 519–522):

```scss
/* Before */
.page__hero--overlay {
  padding-top: 10px !important;
  padding-bottom: 10px !important;
}

/* After — higher specificity replaces !important */
body.layout--splash .page__hero--overlay {
  padding-top: 10px;
  padding-bottom: 10px;
}
```

**Result:** ~20 `!important` flags reduced to ~12 (keeping only the ones required for theme overrides on `author__urls` and hidden elements).

---

## Verification

### Step 1: Automated Build Validation (run by Claude Code)

After implementation, run these checks before any manual testing. All must pass.

```bash
# 1. Jekyll build — confirms SCSS compiles and site generates without errors
bundle exec jekyll build --strict_front_matter 2>&1 | tail -5
# Expected: "done in X seconds" with no errors

# 2. Verify no emojis remain in index.html
grep -Pn '[\x{1F300}-\x{1F9FF}]|[\x{2600}-\x{2B55}]|[\x{FE00}-\x{FE0F}]' index.html
# Expected: no output (exit code 1)

# 3. Verify no inline style attributes remain in index.html
grep -n 'style=' index.html
# Expected: no output (exit code 1)

# 4. Verify no !important in heading rules (should have been removed)
grep -n '!important' _sass/custom.scss | grep -E 'font-size|font-weight' | grep -v 'author__urls'
# Expected: no output — all heading !important flags removed

# 5. Verify design tokens are defined at top of file
head -20 _sass/custom.scss | grep '\$accent-color'
# Expected: $accent-color: #007acc;
```

If any check fails, fix the issue before proceeding to manual testing.

### Step 2: Manual Browser Testing

Open the local site (`bundle exec jekyll serve` or `docker-compose up`) and verify:

- [ ] **Homepage — Featured section:** Renders correctly, "Featured Articles" heading has no emoji, cards display with hover lift
- [ ] **Homepage — Recent section:** "Recent Articles" heading has no emoji, grid spacing unchanged
- [ ] **Homepage — Categories:** "Explore by Topic" and all three category titles have no emojis, text-align center still works without inline styles
- [ ] **Featured badge:** Shows "Featured" text on yellow background (no star emoji)
- [ ] **Responsive — 480px:** Hero stacks vertically, cards go single-column, all text readable
- [ ] **Responsive — 768px:** Hero stacks, cards go single-column, category cards stack
- [ ] **Responsive — 1024px:** Hero side-by-side but compact, cards in 2-column-ish layout
- [ ] **Responsive — Desktop:** Full 3-column grids, hero side-by-side
- [ ] **Post page — Headings:** H2 is 1.4em with bottom border, H3 is 1.15em (clearly smaller), H4 has muted color
- [ ] **Post page — Hover states:** Card hover lift and link color transitions work
- [ ] **Homepage — Section headings:** Featured and Recent h2 headings retain `margin: 2em 0 1em 0` spacing (now from CSS rule, not inline)
- [ ] **Homepage — Category section:** Background still light gray, corners now 12px (was 8px inline), vertical padding slightly tighter (1.5em vs 2em) — both intentional
- [ ] **No visual regressions:** No other unexpected layout shifts
