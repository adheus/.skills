# .skills

Reusable [Claude Code](https://claude.ai/code) skills for development.

## Available Skills

### roku

Roku BrightScript and SceneGraph development reference. Covers:

- **BrightScript Language** — Types, operators, control flow, functions, scope, threading, error handling, global functions
- **SceneGraph Framework** — XML elements, component lifecycle, key nodes (Task, Timer, ContentNode, Label, Poster, RowList, MarkupGrid, etc.)
- **Video Node** — Complete spec: playback, DRM, captions, trickplay, buffering, audio tracks, CDN, UI customization
- **SGDEX** — SceneGraph Developer Extensions: GridView, MediaView, DetailsView, ContentHandler, theming, navigation
- **Community Tools** — BrighterScript, ropm, bslint, Rooibos

## Installation

Copy the skill folder into your Claude Code skills directory:

```bash
# Global (all projects)
cp -r roku/ ~/.claude/skills/roku/

# Project-level (single repo)
cp -r roku/ <your-repo>/.claude/skills/roku/
```

## Usage

The skill auto-activates when working with `.brs` or `.xml` files. You can also invoke it manually:

```
/roku [topic or question]
```
