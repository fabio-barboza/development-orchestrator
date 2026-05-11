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

> **Applicability**: applies ONLY to UI code (frontend / mobile). For backend code, this section is N/A.

The concrete styling methodology is declared by the project in the Tech Spec's "Implementação Visual" section — could be CSS variables + BEM, utility-first / Tailwind, CSS Modules, CSS-in-JS, native StyleSheet, Flutter `ThemeData`, etc. The principles below apply regardless of methodology; specific syntax follows the project's choice.

- **Design tokens over literals**: themeable values (colors, spacing, radii, typography) MUST reference the project's tokens (CSS variables, theme keys, design system tokens, platform tokens) rather than literal values.
- **Style coverage**: every class / selector / style key referenced in UI code MUST have a corresponding rule defined.
- **Adaptive layout**: respect the strategy declared in the PRD / Tech Spec — mobile-first breakpoints (web) or platform adaptive conventions (mobile).
- **Naming convention**: follow the methodology declared in the Tech Spec; do not enforce BEM, utility-first, or any specific convention.
- **Design token consistency**: use the same token names across components; do not redefine duplicates.
- **Accessibility primitives**: define focus indicators (web: `:focus-visible`; mobile: platform focus rings); respect contrast ratios from the PRD's WCAG target.
