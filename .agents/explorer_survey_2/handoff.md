# Handoff Report — Explorer 2: Brand & Platform Visual Signatures, Data Model & Lookups

**Task**: Survey Scope on 4 Sushi Brands, 5 Delivery Platforms, Data Model, Database Lookups, Color Coding Tokens, UI Placement, and R2 Implementation Blueprint.  
**Author**: Explorer 2  
**Target Path**: `.agents/explorer_survey_2/handoff.md`  
**Handoff Type**: Hard (Investigation & Survey Complete)

---

## 1. Observation

1. **Brand & Platform Data Models (`prisma/schema.prisma`)**:
   - `model Brand` (lines 89–98) defines `id`, `name` (unique), `isActive`, `createdAt`, `updatedAt`, `categories Category[]`, `orders Order[]`.
   - `model Platform` (lines 100–107) defines `id`, `name` (unique), `isActive`, `createdAt`, `orders Order[]`.
   - `model Order` (lines 187–188, 219–220) defines relations `platform Platform @relation(fields: [platformId], references: [id])` and `brand Brand @relation(fields: [brandId], references: [id])`, indexed at `@@index([platformId, brandId])` (line 233).
   - Neither model stores styling, HEX codes, or icons in the PostgreSQL database.

2. **Lookup Services & Auto-Seeding**:
   - `src/services/lookups.ts` (lines 23–44) automatically seeds the 5 delivery platforms if the table is empty:
     `const defaultNames = ["Phone", "Talabat", "elmenus", "InstaShop", "HarryApp"];`
   - `src/services/menu.ts` (lines 21–27) queries active brands:
     `prisma.brand.findMany({ where: { isActive: true }, orderBy: { name: "asc" }, select: { id: true, name: true } })`.

3. **Current UI Rendering of Brands & Platforms**:
   - `src/components/orders/orders-table.tsx` (lines 620–629):
     ```tsx
     <Badge variant="secondary" className="text-[11px] font-semibold">
       {order.brand.name}
     </Badge>
     <span className="text-[11px] text-muted-foreground">
       {order.platform.name}
     </span>
     ```
   - `src/components/orders/orders-table.tsx` (lines 874–882): Mobile card renders `<Badge variant="secondary">{order.brand.name}</Badge>` and `<Badge variant="outline">{order.platform.name}</Badge>`.
   - `src/components/orders/order-details-modal.tsx` (lines 232–237): Renders generic `<Badge variant="secondary">{order.brand.name}</Badge>` and `<Badge variant="outline">{order.platform.name}</Badge>`.
   - `src/components/orders/order-form.tsx` (lines 468–493, 502–516): Brand selection uses standard uncolored buttons (`variant={isSelected ? "default" : "outline"}`) and platform selection uses standard uncolored `<Select>`.

4. **Global Theme Configuration (`src/app/globals.css`)**:
   - Uses Tailwind CSS v4 (`@import "tailwindcss";`, `@import "tw-animate-css";`, `@import "shadcn/tailwind.css";`).
   - Dark mode configured via `@custom-variant dark (&:is(.dark *));`.
   - All colors derive from semantic OKLCH CSS variables (`--primary`, `--secondary`, `--muted`, `--accent`, `--destructive`, `--border`).

---

## 2. Logic Chain

1. **Observation 1 & 2** show that the backend stores Brand and Platform as dynamic normalized database entities without UI styling properties. This means styling and visual tokens belong in a decoupled frontend design token layer rather than altering the Prisma schema.
2. **Observation 3** reveals that all current UI components render brands and platforms using monochrome `variant="secondary"` or `variant="outline"` badges. This directly creates visual ambiguity during high-stress kitchen operations where quick scanning is required across 4 brands and 5 platforms.
3. **Observation 4** indicates the application is powered by Tailwind v4 with `@custom-variant dark`. Therefore, our visual token registry must supply Tailwind classes that function in both light mode and dark mode without requiring separate CSS build steps.
4. Designing mutually distinct color families for the 4 Brands (Flower = Sakura Pink `#EC4899`, Mastery = Artisan Gold `#D97706`, Niwa = Matcha Green `#059669`, Tobiko = Tobiko Orange `#EA580C`) ensures zero visual collision between brands.
5. Designing authentic color signatures for the 5 Platforms (Talabat = `#FF5A00`, elmenus = `#E22D38`, InstaShop = `#00A887`, HarryApp = `#6366F1`, Phone = `#0284C7`) ensures immediate recognition for couriers and cashiers.
6. Centralizing these mappings in `src/lib/visualTokens.ts` and encapsulating them into `BrandBadge` (`src/components/ui/brand-badge.tsx`) and `PlatformBadge` (`src/components/ui/platform-badge.tsx`) creates a clean single source of truth that will immediately empower `orders-table.tsx`, `order-form.tsx`, `order-details-modal.tsx`, and the upcoming Kitchen Kanban (`kitchen-kanban.tsx`).

---

## 3. Caveats

- **No DB Migration Required**: Adding visual tokens does not require running Prisma migrations because lookups are matched by brand/platform name.
- **Normalization Required**: Brand and platform names in database instances might have casing or spacing differences (e.g. "Flower" vs "Flower Sushi" vs "flower", or "elmenus" vs "Menus"). The lookup helper must normalize strings via case-insensitive trim matching.
- **Third-Party Logistics**: If new platforms or brands are added dynamically via admin UI in the future, the fallback should gracefully render a clean neutral token rather than throwing an error.

---

## 4. Conclusion

The data model and lookups for the 4 sushi brands and 5 delivery platforms are solidly established in the backend, but the frontend currently lacks the visual identity necessary for quick-glance kitchen ergonomics.

The recommended R2 implementation will introduce:
1. `src/lib/visualTokens.ts` as the single source of truth for brand and platform visual tokens with light/dark mode classes, HEX values, and Japanese Kanji glyphs.
2. `src/components/ui/brand-badge.tsx` and `src/components/ui/platform-badge.tsx` as standard presentation components.
3. Extensions to `orders-table.tsx`, `order-details-modal.tsx`, `order-form.tsx`, and the future Kitchen Kanban board to render these color-coded visual signatures.

---

## 5. Verification Method

To independently verify the survey findings and prepare for implementation:
1. Inspect `prisma/schema.prisma` lines 89–107 and 187–220 to verify the Brand, Platform, and Order relationships.
2. Inspect `src/services/lookups.ts` lines 23–44 to confirm default platform seeding.
3. Inspect `src/components/orders/orders-table.tsx` lines 620–629 and 874–882 to verify current generic badge rendering.
4. Run `npm run typecheck` and `npm run lint` to verify clean baseline before R2 work begins.
5. Review detailed analysis in `.agents/explorer_survey_2/analysis.md`.
