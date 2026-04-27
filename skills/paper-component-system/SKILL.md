---
name: paper-component-system
description: Use this skill when working on Paper Design projects that have a component library or need one set up. Trigger when the user asks to create, sync, or update design components across artboards or pages, when adding components to new screens, when syncing design changes across multiple pages, when setting up a component system in a new Paper Design project, when a user places components manually and needs them tracked, when exporting Paper designs to production code, or when pushing code/style changes back into Paper artboards. Trigger phrases include "create component", "sync components", "update all buttons", "add to manifest", "component library", "design system", "sync all", or any mention of tracking component instances across a Paper Design file.
---

# Paper Component System

This skill defines a reusable component workflow for Paper Design — covering component creation, instance tracking, multi-page sync, and design↔code exchange. It works alongside the `paper-desktop` MCP server.

## Before Starting

Check for a project `CLAUDE.md` — it will contain:
- Path to this project's `manifest.json`
- Design tokens (primary color, border-radius, font)
- Known artboard node IDs

If no `CLAUDE.md` exists, run **Setup** first.

---

## Setup (New Project)

When starting a component system in a new project:

1. Create `components/` directory and `components/manifest.json`:
```json
{ "components": {} }
```

2. Create a `CLAUDE.md` in the project root using this template:
```markdown
# [Project Name] — Paper Design

## Component System
Uses the `paper-component-system` skill.
Manifest: `components/manifest.json`

## Design Tokens
- Primary: [color]
- Border radius: [value]
- Font: [family]

## Known Artboards
| Artboard | Page | Node ID |
|----------|------|---------|
```

---

## Naming Convention

All component instances in Paper **must** use bracket-prefixed layer names:
- `[Button-Primary]`, `[Card]`, `[NavBar]`, `[Popup-Confirm]`

**Detached instance** (one-off, excluded from future syncs): rename without brackets:
- `Button-Primary-HeroSection`, `Card-Featured`

---

## Creating a Component

1. Call `get_font_family_info` before any typography (once per session)
2. Use `write_html` to place the component on the target artboard
3. Name the root node `[ComponentName]`
4. Save master HTML to `components/ComponentName.html`
5. Register in `manifest.json`:

```json
"ComponentName": {
  "file": "components/ComponentName.html",
  "instances": [
    { "artboard": "ArtboardName", "nodeId": "node-id-from-paper" }
  ]
}
```

---

## Syncing Components

### Single component sync
1. Read `components/ComponentName.html` (the master)
2. Look up all instances in `manifest.json`
3. For style/content changes: use `update_styles` + `set_text_content`
4. For structural changes: delete node, `write_html`, rename, update manifest nodeId

### Global sync (all components, all pages)
Collect every node ID across all instances in `manifest.json` and fire one batch `update_styles` call. Node IDs work cross-page — no page navigation needed.

### Auto-discovery on sync
On every sync, scan all known artboard IDs for unregistered instances:

```
For each unique artboard nodeId referenced in manifest.json:
  Run get_tree_summary(artboardId)
  Find all nodes whose name matches /^\[.+\]$/
  If node matches a known component AND its nodeId is not in the manifest → register it
```

**Limitation:** `get_tree_summary` only sees artboards whose node IDs are already known. A completely new page requires the user to navigate there once so `get_basic_info` can reveal its artboards. After that first visit, auto-discovery works autonomously.

### Multi-page sync
`update_styles`, `set_text_content`, and `rename_nodes` accept node IDs from any page. Always collect all node IDs from the manifest first, then fire one batch call. Never ask the user to navigate pages for a sync.

---

## Design → Code

Use the `/paper-desktop:design-to-code` skill for the full export workflow.

Key rules:
- Always use `get_jsx` + `get_computed_styles` for exact values
- Use `get_fill_image` for background images/fills
- **Never** read measurements or colors from screenshots — screenshots are for visual review only
- Match the conventions of the target codebase (CSS variables, utility classes, etc.)

---

## Code → Design

When a design token changes (color, spacing, border-radius, font):
1. Update the master `components/*.html` file(s)
2. Update state tokens in `manifest.json`
3. Run a global sync — all instances update in one batch call

---

## Manifest Schema

```json
{
  "components": {
    "ComponentName": {
      "file": "components/ComponentName.html",
      "states": {
        "default":  { "background": "#111111" },
        "hover":    { "background": "#222222" },
        "pressed":  { "background": "#333333" },
        "disabled": { "background": "#F3F4F6", "color": "#9CA3AF" },
        "loading":  { "background": "#111111", "opacity": "0.8" }
      },
      "instances": [
        {
          "artboard": "ArtboardName",
          "nodeId": "node-id",
          "note": "optional — e.g. label override: Save"
        }
      ]
    }
  }
}
```

---

## Manifest Maintenance

**After placing a component manually in Paper:**
Tell Claude: *"I added a `[ComponentName]` to the [Page] artboard"* — Claude will navigate there, find the node, and register it.

**Detaching an instance:**
1. Rename the node in Paper from `[ComponentName]` to `ComponentName-ScreenName`
2. Remove that entry from `manifest.json`

**Stale node IDs** (node was deleted and recreated):
Run `get_tree_summary` on the artboard, find the new node ID, update `manifest.json`.

---

## Screenshot Verification

`get_screenshot` works on any node ID from any page — no navigation needed. After any sync or edit, always take screenshots of all affected artboards before reporting done. Use the Review Checkpoints from `get_guide({ topic: "paper-mcp-instructions" })`.
