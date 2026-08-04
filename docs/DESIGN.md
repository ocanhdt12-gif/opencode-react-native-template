# Design Specification

> Paste Figma links, design notes, or upload screenshots.
> Agent will extract design tokens and screen specs from this file.

---

## Design References

**Figma Link:** [https://figma.com/file/...]
**Design Tool:** Figma / Sketch / Adobe XD / Other

**Reference Images:**
- [Attach screenshots or paste image URLs]

---

## Brand & Visual Identity

### Color Palette

| Token | Value | Usage |
|-------|-------|-------|
| Primary | #000000 | Buttons, links, CTAs |
| Secondary | #000000 | Supporting elements |
| Background | #ffffff | Page background |
| Surface | #f5f5f5 | Cards, panels |
| Text Primary | #111111 | Headings, body |
| Text Secondary | #666666 | Labels, captions |
| Border | #e0e0e0 | Dividers, inputs |
| Error | #ef4444 | Error states |
| Success | #10b981 | Success states |
| Warning | #f59e0b | Warning states |

### Typography

| Token | Font | Size | Weight | Usage |
|-------|------|------|--------|-------|
| Heading 1 | | 36px | 700 | Page titles |
| Heading 2 | | 24px | 600 | Section titles |
| Heading 3 | | 20px | 600 | Card titles |
| Body | | 16px | 400 | Body text |
| Small | | 14px | 400 | Labels, captions |
| XSmall | | 12px | 400 | Meta, hints |

### Spacing Scale

| Token | Value |
|-------|-------|
| xs | 4px |
| sm | 8px |
| md | 16px |
| lg | 24px |
| xl | 32px |
| 2xl | 48px |
| 3xl | 64px |

### Border Radius

| Token | Value | Usage |
|-------|-------|-------|
| sm | 4px | Inputs, tags |
| md | 8px | Cards |
| lg | 12px | Modals |
| full | 9999px | Pills, avatars |

---

## Screen Inventory

List all screens and their key UI elements:

### Screen: [Name] (e.g. Login)
- **Route:** `/login`
- **Layout:** [Centered / Full width / Split]
- **Components:** Logo, Email input, Password input, Submit button, Forgot password link
- **States:** Default, Loading, Error
- **Notes:** [Any special behavior]

### Screen: [Name] (e.g. Dashboard)
- **Route:** `/dashboard`
- **Layout:**
- **Components:**
- **States:**
- **Notes:**

---

## Component Library

| Component | Variants | Notes |
|-----------|----------|-------|
| Button | Primary, Secondary, Danger, Ghost | Sizes: sm, md, lg |
| Input | Default, Error, Disabled | With/without label |
| Card | Default, Hoverable, Selected | |
| Modal | Small, Medium, Large | |
| Table | Basic, Sortable, Paginated | |
| Badge | Info, Success, Warning, Error | |

---

## Design Style

- **Overall feel:** [Minimal / Modern / Corporate / Playful]
- **Color scheme:** [Light / Dark / Both]
- **UI Library:** [shadcn/ui / MUI / Tailwind only / Custom]
- **Animation level:** [None / Subtle / Rich]

---

## Responsive Breakpoints

| Breakpoint | Width | Target |
|------------|-------|--------|
| Mobile | < 768px | Phone |
| Tablet | 768px – 1024px | iPad |
| Desktop | > 1024px | Laptop/Desktop |
