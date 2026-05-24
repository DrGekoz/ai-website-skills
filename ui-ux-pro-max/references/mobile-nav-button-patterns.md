# Mobile Nav Button & Responsive Hiding Patterns

> Ref: Ads Doctor Melbourne project — iterative corrections session (May 2026)

## Hamburger Menu Button — Do's and Don'ts

### Placement
- **DO** place the hamburger button **inside the header**, right after the desktop `</nav>` tag, in the same flex row as the brand logo/title
- **DON'T** float it as a separate `fixed` or `absolute` button outside the header — it breaks the header's visual cohesion and creates z-index headaches

### Icon Style
- **DO** use a plain 3-line hamburger SVG icon (`<line x1="3" y1="12" ...>`)
- **DON'T** use an image asset (webp/png) as the hamburger icon — SVG scales cleanly at any size
- **DON'T** wrap the icon in a circular button (no `rounded-full`, no `w-12 h-12` fixed box, no border/shadow)
- **DON'T** use the OrangeIcons/205 dropdown-menu-icon for the hamburger — that's a desktop nav indicator, not a mobile menu trigger

### Color & Contrast
- **DO** use an explicit brand color for the icon stroke (e.g. `stroke="#E86A28"`) — must be visible on the header background
- **DON'T** use `stroke="white"` on a light/white header background — invisible
- **DON'T** rely on `stroke="currentColor"` — it inherits the surrounding text color which may be low-contrast on mobile

### Toggle Pair
- **DO** provide both `#hamburgerIcon` and `#closeIcon` SVGs inside the same button
- **DO** mark the close icon as `class="hidden"` initially
- **DO** toggle both icons in the JS: `hamburgerIcon.classList.toggle('hidden')` / `closeIcon.classList.toggle('hidden')`
- **DON'T** omit the close icon — the user needs a visual way to close the menu

## Responsive Hiding Patterns

### Hide on mobile, show on desktop
```html
<div class="hidden md:block">...</div>   <!-- or md:flex, md:grid -->
```

- `hidden` alone hides on ALL screen sizes (not useful for responsive)
- `hidden md:block` hides on mobile (<768px), shows as block on desktop (≥768px)
- `hidden md:flex` hides on mobile, shows as flex on desktop

### Show on mobile, hide on desktop
```html
<div class="md:hidden">...</div>
```

### Common mistake
Using `class="hidden"` with no responsive prefix when the intent is "hide on mobile only". Always pair with an `md:*` breakpoint to restore visibility on desktop.

## Mobile Button Centering

On mobile, buttons should be centered horizontally. Add a media query:

```css
@media (max-width: 767px) {
  .btn, .btn-mint, .btn-primary, [class*="btn-"] {
    display: flex !important;
    margin-left: auto !important;
    margin-right: auto !important;
    width: fit-content !important;
    justify-content: center;
  }
}
```

This overrides `inline-flex` to `flex` (block-level), then centers with auto margins. Works for all button-like classes.

## Hamburger vs Floating Button

| Pattern | Header-inline | Floating button |
|---------|---------------|-----------------|
| Placement | Inside `<header>` after `</nav>` | `fixed top-4 right-4` outside header |
| Visual | Inline with brand logo/title | Separate circle in corner |
| Pros | Part of header layout, consistent | None — always worse UX |
| Cons | None | Z-index wars, overlaps content, separate from brand |
| Mobile nav menu title alignment | Hamburger sits next to "Menu" or brand name | No title context |

**Always use header-inline. Never use a floating button for the mobile nav trigger.**
