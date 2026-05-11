# Code Standards Reference

## Naming Conventions
- **camelCase**: methods, functions, variables
- **PascalCase**: classes, interfaces
- **kebab-case**: files, directories

## Code Rules
- All code in English (variables, functions, classes, comments)
- No abbreviations, no names over 30 characters
- No magic numbers — use named constants
- Functions start with a verb, perform single clear action
- Maximum 3 parameters per function (use objects for more)
- Functions do mutation OR query, never both
- Maximum 2 nesting levels for conditionals, prefer early returns
- Never use boolean flag parameters to toggle behavior
- Maximum 50 lines per method
- Maximum 300 lines per class
- No blank lines within methods/functions
- Avoid comments — code should be self-explanatory
- One variable per line, declare close to usage

## Visual / UI Styling

> **Applicability**: this section applies ONLY when the task modifies UI files in a visual project (frontend, mobile, full-stack). For backend-only tasks/projects, this entire section is **N/A** and must be skipped silently. Visual issues are ALWAYS documented in the review. Residual severity after `visual_retry = 2/2` determines whether the pipeline continues (MAIOR/MENOR → continue) or halts (CRÍTICO → halt, parallel to failing tests). See `do-execute-task` Step 6.5 for the full rule.

The concrete styling methodology is **declared by the project** in the Tech Spec ("Implementação Visual" section) — could be CSS variables + BEM, utility-first/Tailwind, CSS Modules, CSS-in-JS, native StyleSheet (React Native / iOS / Android), Flutter `ThemeData`, etc. The principles below apply regardless of methodology; the specific syntax follows the project's choice.

- **Design tokens over literals**: themeable values (colors, spacing, radii, typography) MUST reference the project's tokens (CSS variables, theme keys, design system tokens, platform tokens), not literal values. Hardcoded values are flagged in review as MENOR (isolated) or MAIOR (when theming is in scope).
- **Style coverage rule (symmetry by severity)**: every class / selector / style key referenced in UI code MUST have a corresponding rule defined. When gaps are detected, the executor runs the ACTIVE RETRY LOOP defined in `do-execute-task` Step 6.5 — up to 2 dedicated retries to close the gaps (generate missing rules, replace hardcoded literals with tokens, add adaptive rules). After `visual_retry = 2/2`:
  - Residual **MAIOR/MENOR** gaps (1–3 classes faltando OR literais hardcoded isolados) → documented and pipeline continues. Cosmético segue.
  - Residual **CRÍTICO** gaps (4+ classes/seletores sem regra, OR component visually broken, OR theme support exigido e não aplicado) → HALT the pipeline, parallel to a failing test. Reported clearly to the user; next task does NOT begin until resolved.
- **Adaptive layout**: respect the strategy declared in the PRD/Tech Spec.
  - Web: mobile-first when so declared (base styles for the smallest viewport, `@media (min-width: ...)` to scale up).
  - Mobile: handle screen-size variation, orientation, and tablet vs. phone per platform conventions.
- **Naming convention**: follow the methodology declared in the Tech Spec. Do NOT enforce BEM, utility-first, or any specific convention here — the review checks consistency with what the project declared, not a fixed standard.
- **Design token consistency**: use the same token names across components; do not redefine duplicate tokens.
- **Accessibility primitives**: define focus indicators (web: `:focus-visible`; mobile: platform focus rings), respect contrast ratios from the PRD's WCAG target.
