# UI/UX Pro Max - Design Intelligence (Local Reference Dump)

## Key Principles for Sushi Dark Kitchen POS & Order Control
- **Touch & Interaction Ergonomics**: POS terminals and kitchen tablets need >=44x44px touch targets (`min-h-11 min-w-11`), generous padding, and clear active states.
- **Night-Shift Contrast (Obsidian & Amber)**: Deep black background (`oklch(0.08 0.015 250)`), pure white foreground (`oklch(0.99 0 0)`), high-chroma sushi amber accent (`oklch(0.72 0.22 45)`), sharp borders (`oklch(0.30 0.02 250)`).
- **Tabular Numbers**: Use `tabular-nums` for price tags, order numbers, and timers to avoid visual jitter during rapid updates.
- **Bilingual Polish**: 100% Arabic RTL (default) with English LTR support; use logical Tailwind properties (`ms-`, `me-`, `ps-`, `pe-`, `start-`, `end-`) and `rtl:rotate-180` for directional indicators.
- **Accessibility & Focus**: High contrast, visible focus rings, aria labels on icon-only buttons.
