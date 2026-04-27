# paper-workflows

A Claude Code plugin providing reusable component system workflows for [Paper Design](https://paperlayer.com).

## Skills

### `paper-component-system`

Covers the full lifecycle of a Paper Design component library:

- **Setup** — initialise `manifest.json` and `CLAUDE.md` in a new project
- **Create** — write HTML components, name them, register instances in the manifest
- **Sync** — propagate style/content changes to every instance across all pages in one batch call
- **Auto-discover** — find unregistered `[ComponentName]` nodes on known artboards automatically
- **Design → Code** — export exact values via `get_jsx` / `get_computed_styles`
- **Code → Design** — push token changes back into Paper with a global sync

## Installation

Add to `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "paper-workflows": {
      "source": {
        "source": "github",
        "repo": "shankarkrishna8/paper-workflows"
      }
    }
  },
  "enabledPlugins": {
    "paper-workflows@paper-workflows": true
  }
}
```

## Requirements

- [Paper desktop app](https://paperlayer.com) with the MCP server running (`paper-desktop@paper` plugin enabled)
- Claude Code

## Usage

The skill auto-triggers on phrases like "create component", "sync components", "update all buttons", "add to manifest", "design system", or "sync all". You can also invoke it explicitly with `/paper-workflows:paper-component-system`.
