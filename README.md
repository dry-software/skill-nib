# skill-nib

A [Claude Code skill](https://docs.anthropic.com/en/docs/claude-code/skills) that teaches Claude how to perform desktop automation using **nib** (nut.js Instrumentation Bundle).

## What is this?

`.claude/skills/nib/SKILL.md` is a skill file for Claude Code. When installed, it gives Claude detailed knowledge of the `nib` CLI tool, enabling it to automate desktop applications on your behalf — clicking buttons, filling forms, reading screen content, navigating menus, and more.

The skill covers:

- **Window management** — listing, focusing, resizing, and moving windows
- **Accessibility-based interaction** — taking snapshots of UI element trees and interacting with elements by ref (e.g. `@btn:Save`, `@txt:Username`) instead of fragile screen coordinates
- **Mouse & keyboard** — coordinate-based clicking, dragging, typing, and key presses as a fallback
- **Screen reading** — screenshots, OCR text extraction, and color sampling
- **Clipboard, timing, and batch execution**

## Prerequisites

You need `nib` installed and available on your `PATH`.

You can install it via `npm i -g @nut-tree/nib`

## Installation

### Project-level (recommended)

Copy the content of `.claude/` in this repo into your project so the skill file lives at:

```
your-project/
  .claude/
    skills/
      nib/
        SKILL.md
```

Claude Code automatically discovers skills in the `.claude/skills/` directory of your project.

### Global installation

To make the skill available across all projects, place it in your global Claude Code skills directory:

```
~/.claude/skills/nib/SKILL.md
```

## Usage

Once installed, invoke the skill from Claude Code:

```
/nib open the Settings window in System Preferences and enable Dark Mode
```

or simply describe a desktop automation task and Claude will use `nib` to carry it out.

