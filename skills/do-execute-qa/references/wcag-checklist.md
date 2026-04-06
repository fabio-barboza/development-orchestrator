# WCAG 2.2 Accessibility Checklist

Quick reference for browser-based accessibility verification (Level AA).

## Keyboard Navigation
- [ ] All interactive elements reachable via Tab
- [ ] Shift+Tab reverses focus order correctly
- [ ] Enter/Space activate buttons and links
- [ ] Escape closes modals, dropdowns, and overlays
- [ ] Focus never gets trapped (unless intentional modal)
- [ ] Focus order is logical and follows visual layout

## Focus Visibility
- [ ] Focus indicator is visible on all focusable elements
- [ ] Focus indicator has sufficient contrast (min 3:1 ratio against adjacent colors)

## Labels & Semantics
- [ ] All form inputs have associated `<label>` or `aria-label`
- [ ] Buttons and icon-only controls have descriptive text or `aria-label`
- [ ] Images have `alt` text (empty `alt=""` for decorative images)
- [ ] Page has a single `<h1>` and logical heading hierarchy
- [ ] Links are descriptive ("View report" not "Click here")

## Color & Contrast
- [ ] Text contrast ≥ 4.5:1 (normal text) or ≥ 3:1 (large text ≥ 18pt / 14pt bold)
- [ ] UI component contrast ≥ 3:1 (borders, icons, active states)
- [ ] Information is not conveyed by color alone (use icons or text labels too)

## Forms & Errors
- [ ] Required fields are indicated (not only by color)
- [ ] Error messages identify the specific field and describe the problem
- [ ] Success/error states are announced to screen readers (aria-live or role="alert")
- [ ] Input purpose is identifiable (autocomplete attributes where applicable)

## Dynamic Content
- [ ] Status messages use `role="status"` or `role="alert"` (no focus move needed)
- [ ] Modals/dialogs trap focus and return it on close
- [ ] Loading states are communicated (aria-busy or visible text)

## Touch & Pointer
- [ ] Touch targets ≥ 24×24 CSS pixels (WCAG 2.2 minimum)
- [ ] No functionality requires complex gestures (swipe, pinch) without a simple alternative

## How to Test with Browser MCP
1. Use `browser_snapshot` to read semantic structure and verify labels/roles
2. Use `browser_press_key` with `Tab` to navigate and check focus order
3. Use `browser_press_key` with `Enter`/`Escape` to verify keyboard interactions
4. Use `browser_take_screenshot` to visually confirm focus indicators and contrast
5. Use `browser_console_messages` to check for accessibility-related warnings
