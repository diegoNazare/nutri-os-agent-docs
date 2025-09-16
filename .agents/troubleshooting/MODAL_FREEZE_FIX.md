# Modal Freeze Fix Guide

This document describes the definitive solution for cases where the interface "freezes" (becomes unclickable, unscrollable, or loses focus) after opening and closing modals from dropdown menus.

## Root Cause
The freezing occurs due to **focus scope conflicts** between dropdown menus and modals. When a modal opens while a dropdown menu is still mounted, Radix UI blocks `aria-hidden` on elements that retain focus, leaving the page in an unresponsive state.

Console error indicator:
```
Blocked aria-hidden on an element because its descendant retained focus. The focus must not be hidden from assistive technology users.
```

## Solution
**Use `onSelect` with `setTimeout` instead of `onClick` for dropdown menu items that open modals.**

### Implementation Pattern
```tsx
// ❌ WRONG - Causes focus scope conflict
<DropdownMenuItem onClick={() => setShowModal(true)}>
  Open Modal
</DropdownMenuItem>

// ✅ CORRECT - Prevents focus scope conflict
<DropdownMenuItem onSelect={() => setTimeout(() => setShowModal(true), 0)}>
  Open Modal
</DropdownMenuItem>
```

### Why This Works
1. **`onSelect`** ensures the dropdown menu unmounts completely before the modal opens
2. **`setTimeout(..., 0)`** defers modal opening to the next event loop tick
3. **No focus scope overlap** - dropdown is fully unmounted when modal mounts
4. **No `aria-hidden` conflicts** - clean focus management transition

## Application Guidelines
- Apply this pattern to **any** `DropdownMenuItem` that opens a modal, dialog, or alert
- Use `onSelect` instead of `onClick` for dropdown items that trigger modal state changes
- Always wrap the modal opening in `setTimeout(..., 0)` to ensure proper unmounting sequence
- This solution is **more efficient** than page reloads and preserves application state

## Prevention
- Always use `onSelect` for dropdown menu items that open other UI overlays
- Avoid opening modals directly from `onClick` handlers in dropdown contexts
- Test dropdown-to-modal flows to ensure no focus scope conflicts remain

## Status
**SOLVED** - This solution eliminates modal freezing without requiring page reloads or complex cleanup logic.
