# Responsive Tailwind Patterns for Content Sites

Common mobile-responsive patterns used during the Ads Doctor Melbourne redesign.

## Mobile Responsive Class Patterns

| Goal | Classes | Notes |
|------|---------|-------|
| Hide on mobile, show on desktop | `hidden md:block` | Also `hidden md:flex` for flex containers |
| Show on mobile, hide on desktop | `md:hidden` | Keep original as-is |
| Center on mobile, left on desktop | `justify-center md:justify-start` | For flex containers with buttons |
| Responsive padding (mobile tighter) | `p-6 md:p-12` / `px-4 md:px-8` | Always tighten padding on mobile |
| Responsive hero padding (account for sticky header) | `pt-24 pb-8 md:pt-32 md:pb-12` | Mobile ~96px top, desktop 128px |
| Mobile menu max height | `max-h-[70vh] overflow-y-auto` | Prevents menu from going off-screen |

## CSS Media Queries for Global Mobile Rules

Use `@media (max-width: 767px)` (Tailwind's `md` breakpoint is 768px):

```css
@media (max-width: 767px) {
  .btn, .btn-mint, .btn-primary, .btn-secondary, .btn-accent,
  .btn-lg, .btn-outline, .btn-gradient, .btn-orange,
  [class*="btn-"],
  a[href*="calendly"], a[href*="wa.me"] {
    display: flex !important;
    margin-left: auto !important;
    margin-right: auto !important;
    width: fit-content !important;
    justify-content: center;
  }
  .flex-wrap.gap-4 {
    justify-content: center !important;
  }
}
```

## Hamburger Menu Icon Pattern

Use a plain 3-line SVG for the mobile hamburger — no image icons, no circular button wrapper:

```html
<button class="md:hidden" onclick="toggleMobileMenu()" aria-label="Toggle mobile menu">
  <svg id="hamburgerIcon" xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="#E86A28" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
    <line x1="3" y1="12" x2="21" y2="12" /><line x1="3" y1="6" x2="21" y2="6" /><line x1="3" y1="18" x2="21" y2="18" />
  </svg>
  <svg id="closeIcon" class="hidden" xmlns="http://www.w3.org/2000/svg" width="28" height="28" viewBox="0 0 24 24" fill="none" stroke="#E86A28" stroke-width="3" stroke-linecap="round" stroke-linejoin="round">
    <line x1="18" y1="6" x2="6" y2="18" /><line x1="6" y1="6" x2="18" y2="18" />
  </svg>
</button>
```

Key rules:
- Always place the hamburger button INSIDE the header, after `</nav>`, not as a floating element
- Use `stroke="#E86A28"` (burnt orange) for visibility on light backgrounds
- Always include both `hamburgerIcon` and `closeIcon` SVGs with proper IDs
- The close icon should start `hidden` (toggled by JS)

## Toggle Functions

```javascript
function toggleMobileMenu() {
  const menu = document.getElementById('mobileMenu');
  const hamburgerIcon = document.getElementById('hamburgerIcon');
  const closeIcon = document.getElementById('closeIcon');
  if (!menu) return;
  menu.classList.toggle('hidden');
  if (menu.classList.contains('hidden')) {
    if (hamburgerIcon) hamburgerIcon.classList.remove('hidden');
    if (closeIcon) closeIcon.classList.add('hidden');
  } else {
    if (hamburgerIcon) hamburgerIcon.classList.add('hidden');
    if (closeIcon) closeIcon.classList.remove('hidden');
  }
}
function toggleMobileDropdown(button) {
  const content = button.nextElementSibling;
  if (!content) return;
  content.classList.toggle('hidden');
  const arrow = button.querySelector('svg');
  if (arrow) arrow.classList.toggle('rotate-180');
}
```

## Pitfalls

- **SVG split across lines**: When using `sed` to batch-replace SVG attributes, the pattern may not match if `stroke="white"` is on a different line than `<svg ...>`. Use a global grep to find remaining `stroke="white"` after batch edits.
- **`hidden` vs `hidden md:block`**: `hidden` hides on ALL screen sizes. Always specify the responsive breakpoint (`hidden md:block`) when you want to hide only on mobile.
- **Floating buttons**: Don't use `position: fixed` floating buttons for mobile nav — place the hamburger icon inside the header's flex container so it's inline with the title.
- **Min-height 100vh**: On mobile, `100vh` doesn't account for browser chrome. Consider `min-h-dvh` or `100dvh` for full-screen sections.
