# Code Standards Reference

**Precedence:** the conventions of the project always win. Take them from the project's instructions file (CLAUDE.md, AGENTS.md or equivalent — read-only), from the `Project Conventions` section of `.do/context.md`, from the project's linter/formatter configuration, and from the existing code. The rules below are language-neutral defaults, applied only where the project defines nothing different.

## Naming Conventions
- Follow the identifier and file-naming conventions of the language and of the project (casing, separators, file layout) as found in the existing code.
- Use the same human language for identifiers and comments that the project already uses.

## Code Rules
- No abbreviations, no names over 30 characters
- No magic numbers — use named constants
- Functions start with a verb, perform single clear action
- Maximum 3 parameters per function (group them into a single structure for more)
- Functions do mutation OR query, never both
- Maximum 2 nesting levels for conditionals, prefer early returns
- Never use boolean flag parameters to toggle behavior
- Maximum 50 lines per function/method
- Maximum 300 lines per class/module
- No blank lines within functions
- Avoid comments — code should be self-explanatory
- One variable per line, declare close to usage
