## 2026-08-26T18:30:17Z
You are Explorer 1 on the Sushi Dark Kitchen Order Control System survey team.
Your working directory is: d:\programming\Projects\Orders Management System\.agents\explorer_survey_1

Inputs to read:
- d:\programming\Projects\Orders Management System\ORIGINAL_REQUEST.md
- d:\programming\Projects\Orders Management System\AGENTS.md

Your Survey Scope: Navigation Header, Dashboard Layout, Theme Switching, and i18n.
Investigate the current codebase for:
1. Header & Navigation structure: Look at existing header/navbar in src/app/[locale]/(dashboard)/..., src/components/, layout files, active route highlighting, route pills, role badges, user profile,  + New Order CTA button.
2. Dark Mode & Theme system: How themes (light/dark/kitchen night shift) are implemented or can be cleanly added (Tailwind v4 theme config, 
ext-themes or custom CSS classes, high contrast kitchen dark mode).
3. i18n & Layout alignment: Next.js 16 i18n setup (src/i18n/, src/messages/ar.json, src/messages/en.json, src/proxy.ts), RTL/LTR ergonomics, touch targets (>=44x44px).
4. Identify all files to be modified/created, potential pitfalls (e.g. PowerShell [locale] escaping, Next 16 middleware in src/proxy.ts), and recommended implementation details for R1, R5, R6.
