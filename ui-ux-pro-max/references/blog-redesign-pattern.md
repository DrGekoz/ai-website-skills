# Blog Redesign Pattern — Ads Doctor Melbourne

**Context**: Complete redesign of an agency blog listing page using UI/UX Pro Max rules directly (scripts unavailable on Windows).

**Product**: Digital marketing agency blog (10 priority categories, ~30 articles, 5 category filters)
**Target**: Melbourne SMB owners
**Brand**: Mint/teal, surgical precision, direct-response (Oswald headings + Inter body)

## Before → After Summary

| Aspect | Before | After |
|--------|--------|-------|
| Hero | Text-only heading | Split-layout featured post card (image + body) |
| Cards | Generic Unsplash, no hierarchy | Consistent 16:9 aspect ratio, category pill overlay, source/read-time footer |
| Filter | Inline color styles per button | Clean `.filter-pill` class with CSS custom properties |
| Search | None | Debounced live-search (title, excerpt, source) |
| Loading | Spinner | Shimmer skeleton (3 cards matching grid) |
| CTA | Single button | Stats row (3 metrics) + dual CTA buttons |
| Animation | None | Staggered fade-in-up, hover lift/zoom, spring curves |
| Accessibility | No skip link, no ARIA | Skip link, aria-selected, aria-controls, aria-current, role=tablist |
| Code quality | Inline styles everywhere | CSS custom properties, reusable classes |

## Architecture That Worked

```
Featured Post (top, split grid) → Filter Bar + Search → Grid (3 col) → Show More → CTA Stats → Footer
```

### Key Design Decisions

1. **Featured post as hero**: Grid card takes first item from filtered list. When filter is active, the featured post still shows (it's just the first filtered result). This preserves visual hierarchy without a separate featured flag.

2. **Reactive filtering**: Filters and search both call `getFiltered()` — filters reset `displayedCount = 9`, search debounces 250ms. Both use the same rendering pipeline.

3. **Card footer pattern**: Source + read time on left, "Read →" arrow on right. The arrow translates +4px on card hover (cheap, effective affordance).

4. **Skeleton loading**: Pure CSS shimmer with `background-size: 200%` animation. Hidden once JS renders. No external dependencies.

## Code Patterns to Replicate

```css
/* Card hover with spring curve */
.blog-card:hover {
  transform: translateY(-3px);
  box-shadow: var(--shadow-md);
  border-color: var(--color-mint);
}

/* Shimmer skeleton */
.skeleton {
  background: linear-gradient(90deg, var(--color-border-light) 25%, ...);
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
}

/* Search debounce */
let searchTimeout;
input.addEventListener('input', () => {
  clearTimeout(searchTimeout);
  searchTimeout = setTimeout(() => { renderAll(); }, 250);
});
```

## Pitfalls Avoided

- **Duplicate mobile menu button**: Original had both a header hamburger AND a floating FAB. Removed the floating one.
- **Inline color maintenance**: Original filters had per-button hex colors. Replaced with a single `.filter-pill.active` class.
- **Unused font loading**: Original loaded Fredoka and Futura. Trimmed to Oswald + Inter only.
- **Bad AI artifact `<!, TEXT , >`**: Checked output with `grep` — none present.

## Verification Checklist

- [ ] Skip link appears on focus
- [ ] Filter pills toggle active state correctly
- [ ] Search reduces results in real-time
- [ ] Featured post updates correctly when filtered
- [ ] Skeleton replaces with real content (DOMContentLoaded)
- [ ] Reduced motion disables all transforms/animations
- [ ] Cards maintain 3-col grid at 1024px+, 2-col at 768px
- [ ] Category pill is readable over images (glass backdrop)
- [ ] Show More button hides when all posts shown
- [ ] Empty state shows with "View All" reset
