# Accessibility Checklist

Baseline reference for accessibility verification of a user interface (target level: AA of the accessibility standard that applies to the interface type). If the PRD, the TechSpec or the project conventions name a specific standard, that standard takes precedence over this list.

## Keyboard / Alternative Input Navigation
- [ ] All interactive elements are reachable without a pointing device
- [ ] Reverse navigation restores the previous focus order correctly
- [ ] The confirm/activate keys operate buttons and links
- [ ] The cancel/escape key closes dialogs, menus and overlays
- [ ] Focus never gets trapped (unless it is an intentional modal)
- [ ] Focus order is logical and follows the visual layout

## Focus Visibility
- [ ] A focus indicator is visible on all focusable elements
- [ ] The focus indicator has sufficient contrast (min 3:1 ratio against adjacent colors)

## Labels & Semantics
- [ ] All form inputs have an associated accessible name (visible label or programmatic label)
- [ ] Buttons and icon-only controls have a descriptive accessible name
- [ ] Images have a text alternative (empty alternative for purely decorative images)
- [ ] Each screen has one main heading and a logical heading hierarchy
- [ ] Links are descriptive ("View report" not "Click here")

## Color & Contrast
- [ ] Text contrast ≥ 4.5:1 (normal text) or ≥ 3:1 (large text)
- [ ] UI component contrast ≥ 3:1 (borders, icons, active states)
- [ ] Information is not conveyed by color alone (use icons or text labels too)

## Forms & Errors
- [ ] Required fields are indicated (not only by color)
- [ ] Error messages identify the specific field and describe the problem
- [ ] Success/error states are announced to assistive technology
- [ ] Input purpose is identifiable (autofill hints where applicable)

## Dynamic Content
- [ ] Status messages are announced without moving focus
- [ ] Dialogs trap focus and return it on close
- [ ] Loading states are communicated (visible text or an announced busy state)

## Touch & Pointer
- [ ] Touch targets are at least 24×24 logical pixels
- [ ] No functionality requires complex gestures (swipe, pinch) without a simple alternative

## How to Test with the UI-testing MCP
1. Use the tool that reads the structure of the screen to verify names, labels and roles
2. Use the key-press tool to navigate by keyboard and check focus order
3. Use the key-press tool with the confirm/cancel keys to verify keyboard interactions
4. Use the screenshot tool to visually confirm focus indicators and contrast
5. Use the console/log tool, if available, to check for accessibility-related warnings
