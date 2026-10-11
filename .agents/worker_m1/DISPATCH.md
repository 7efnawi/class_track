## 2026-08-26T18:39:31Z

You are the Implementation Worker for Milestone 1: Modern Navigation Header, Japanese Branding, Dark Mode & Kitchen Night-Shift Theme System, and Touch Ergonomics.
Your working directory is: d:\programming\Projects\Orders Management System\.agents\worker_m1

Inputs:
- d:\programming\Projects\Orders Management System\ORIGINAL_REQUEST.md
- d:\programming\Projects\Orders Management System\PROJECT.md
- d:\programming\Projects\Orders Management System\AGENTS.md
- d:\programming\Projects\Orders Management System\.agents\explorer_survey_1\analysis.md
- d:\programming\Projects\Orders Management System\.agents\explorer_survey_1\handoff.md

Your Assigned Scope & Exclusive Write Ownership:
1. `src/components/theme-provider.tsx` — Create NextThemesProvider wrapper with `attribute="class"`, `defaultTheme="system"`, `enableSystem`, `themes={["light", "dark", "kitchen"]}`.
2. `src/components/theme-switcher.tsx` — Create interactive dropdown/toggle component with icons (Sun for Light, Moon for Dark, Flame/ChefHat for Kitchen Night-Shift) supporting RTL/LTR.
3. `src/components/layout/dashboard-header.tsx` — Create client component featuring:
   - Japanese sushi dark kitchen branding with logo icon and subtitle.
   - Active-route highlighted pills using `usePathname()` from `@/i18n/navigation`.
   - Persistent, high-visibility "+ New Order" CTA button routing to `/orders/new` with prominent accent styling.
   - Role-based nav links filtering (Cashiers see Orders, Closing, Expenses; Owners/Managers also see Menu, Delivery).
   - Distinct user profile area with role badge.
   - Embedded `LanguageSwitcher` and `ThemeSwitcher`.
   - Mobile responsive drawer/sheet for tablet and phone screens.
4. `src/app/globals.css` — Configure `.kitchen` specialized high-contrast night-shift theme tokens:
   - Deep obsidian background: `oklch(0.08 0.015 250)`
   - Pure crisp foreground: `oklch(0.99 0 0)`
   - High-chroma sushi amber accent: `oklch(0.72 0.22 45)`
   - Distinct card/border delineation tokens.
   - Update `@custom-variant dark (&:is(.dark *, .kitchen *));`.
5. `src/app/[locale]/layout.tsx` — Add `suppressHydrationWarning` to `<html>` and wrap body in `<ThemeProvider>`.
6. `src/app/[locale]/(dashboard)/layout.tsx` — Replace static header with `<DashboardHeader user={user} />`.
7. `src/components/ui/button.tsx` — Add `size="touch"` (`min-h-11 min-w-11 px-5 text-base`) for POS touch targets >=44x44px.
8. `src/messages/ar.json` & `src/messages/en.json` — Add translation keys for nav links, theme names, new order CTA, branding.
9. `PROJECT_LOG.md` — Append changelog entry for Milestone 1 per AGENTS.md Rule 1.

MANDATORY INTEGRITY WARNING:
DO NOT CHEAT. All implementations must be genuine. DO NOT hardcode test results, create dummy/facade implementations, or circumvent the intended task. A teamwork_preview_auditor will independently verify your work. Integrity violations WILL be detected and your work WILL be rejected.

Verification Requirements:
- Run `npm run typecheck` (`tsc --noEmit`) and ensure 0 errors.
- Run `npm run lint` and ensure 0 errors.
- Document all modified files and verification results in `d:\programming\Projects\Orders Management System\.agents\worker_m1\handoff.md`.

When complete, send a message to parent summarizing your work and results.
