# Accessibility proposal — Formula (Excel Formula Editor) v0.5

Basic accessibility requirements for Microsoft Marketplace certification: keyboard focus in the task pane and readable contrast.

Reference: [Accessibility guidelines for Office Add-ins](https://learn.microsoft.com/en-us/office/dev/add-ins/design/accessibility-guidelines).

## Current state (v0.5)

The task pane already has a solid foundation:

- Regions use `aria-label` (formula, catalog, help).
- Both trees use `role="tree"`.
- Dividers use `role="separator"`, `aria-orientation="horizontal"`, and `tabindex="0"`.
- Help region has `aria-live="polite"`.
- Focus-visible styles exist on dividers and some interactive elements.
- Color tokens are defined in CSS (`--ink`, `--muted`, `--green`, `--focus`, etc.).

Gaps for Marketplace “accessibility basics”:

- No documented keyboard model for the trees.
- Limited visible focus on tree nodes, labels, and action buttons.
- Contrast of some muted / error states is not explicitly verified.
- No high-contrast / Windows contrast theme notes.

## Keyboard navigation

### Global Tab order

Recommended order (top to bottom):

1. Formula tree (first focusable node or the tree itself).
2. Formula region divider (`#divider-formula`).
3. Catalog tree.
4. Help region divider (`#divider-help`).
5. Help content (if it contains links or buttons).

All interactive controls inside a region must be reachable by Tab or by tree navigation (see below).

### Tree navigation (formula tree and catalog)

Both trees should follow the common ARIA tree pattern:

| Key | Action |
|-----|--------|
| **Arrow Down** | Move focus to next visible node (sibling or first child). |
| **Arrow Up** | Move focus to previous visible node. |
| **Arrow Right** | Expand collapsed node; if already expanded, move to first child. |
| **Arrow Left** | Collapse expanded node; if already collapsed, move to parent. |
| **Enter / Space** | Activate the node (select for help, start edit on a parameter leaf, or toggle expand). |
| **Home** | Move to first visible node in the tree. |
| **End** | Move to last visible node in the tree. |

Implementation notes:

- Set `tabindex="0"` on the currently focused node and `tabindex="-1"` on others (roving tabindex).
- Keep `role="treeitem"` (or equivalent) on each node row.
- When a parameter leaf enters edit mode (`.value-edit` / `.ref-input`), focus the input and restore tree focus on Escape or after confirm/clear.
- Dividers already have `tabindex="0"`. Arrow Up/Down on a focused divider can optionally resize the adjacent region (or keep only mouse/touch for resize and use Tab to leave).

### Action buttons

- Confirm / clear / remove buttons must be in the Tab order when visible, or reachable via the focused node (e.g. Enter opens edit, then Tab moves to confirm).
- Provide clear accessible names (`aria-label` or visible text).

### Escape and focus restoration

- Escape inside an open editor cancels the edit and returns focus to the node.
- Opening the task pane should place focus on the formula tree (or the first meaningful control).

## Contrast and visibility

### Current palette (approximate ratios)

| Foreground | Background | Approx. ratio | Note |
|------------|------------|---------------|------|
| `#201f1e` (ink) | `#ffffff` | ~16:1 | Excellent |
| `#605e5c` (muted) | `#ffffff` | ~5.7:1 | Passes AA normal text |
| `#107c41` (green) | `#ffffff` | ~5.2:1 | Passes AA |
| `#a4262c` (error) | `#ffffff` / `#fde7e9` | >4.5:1 | Passes |
| `#0f6cbd` (focus) | various | — | Focus ring is blue, good |

### Recommendations

- Keep text contrast ≥ 4.5:1 for normal text and ≥ 3:1 for large text / UI components (WCAG AA).
- Ensure the focus indicator (`outline` or `box-shadow`) has ≥ 3:1 contrast against adjacent colors.
- Do not rely on color alone for meaning:
  - Type errors already use background + inset bar + color — good.
  - Selected nodes use green background + left bar — good.
- Test with Windows high-contrast themes (Aquatic, Desert, Dusk, Night sky) and Office themes. Prefer system colors or CSS custom properties that respond to `prefers-contrast` / Office theming APIs when practical.

### CSS additions (suggested)

```css
/* Stronger, consistent focus */
.label:focus-visible,
.cat-label:focus-visible,
.twist:focus-visible,
.confirm-value:focus-visible,
.clear-value:focus-visible,
.remove-arg:focus-visible,
.ref-input:focus-visible {
  outline: 2px solid var(--focus);
  outline-offset: 1px;
}

/* Optional high-contrast support */
@media (prefers-contrast: more) {
  :root {
    --line: #000000;
    --muted: #000000;
  }
  .label.selected, .cat-label.selected {
    outline: 2px solid currentColor;
  }
}
```

## ARIA improvements

- Ensure every tree node has an accessible name (function name, parameter name, or operator).
- Use `aria-expanded` on collapsible nodes.
- Use `aria-selected` on the currently selected node (in addition to the `.selected` class).
- When a parameter is in an error state, expose it via `aria-invalid="true"` and a short `aria-describedby` pointing to the reason in the help region.
- Keep the existing `aria-live="polite"` on the help region so screen readers announce help changes.

## Testing checklist (for certification notes)

- [ ] Full task pane usable with keyboard only (no mouse).
- [ ] Tab order is logical (formula → divider → catalog → divider → help).
- [ ] Arrow keys navigate both trees; Enter/Space activates.
- [ ] Focus is always visible (focus-visible styles).
- [ ] Contrast of text, selected states, and errors meets AA.
- [ ] Screen reader (Narrator or NVDA) announces tree structure, selection, and help updates.
- [ ] Windows high-contrast theme: all controls remain visible and operable.
- [ ] No keyboard traps; Escape exits edit mode.

## Implementation priority

1. **High** — Roving tabindex + arrow-key navigation on both trees.
2. **High** — Consistent `:focus-visible` styles on all interactive controls.
3. **Medium** — `aria-expanded` / `aria-selected` / `aria-invalid`.
4. **Medium** — Document the keyboard model in `docs/` and mention it in certification notes.
5. **Low** — High-contrast theme tweaks and Office theming API integration.

## Notes for Partner Center certification

> Keyboard: The task pane supports full keyboard operation. Tab moves between the formula tree, resizers, and catalog. Arrow keys navigate tree nodes; Enter/Space selects or edits. Escape cancels an in-progress parameter edit.  
> Contrast: Text meets WCAG AA (4.5:1). Selected and error states use more than color (background + bar). Focus is indicated by a visible outline.

---

*Prepared for Formula / Excel Formula Editor marketplace readiness (v0.5).*
