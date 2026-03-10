You are performing desktop automation using `nib` (nut.js Instrumentation Bundle).

Execute the following task: $ARGUMENTS

---

# How to use nib

Run commands via Bash: `nib <command> [options]`

All commands return JSON to stdout. Always parse and check the `ok` field:
```json
{"ok":true, "command":"...", "data":{...}}
{"ok":false,"command":"...", "error":{"code":"...","message":"...","suggestion":"..."}}
```

If a command fails, read the error message and suggestion carefully before retrying.

---

# Workflow

**Always prefer accessibility-based element interaction over coordinate-based mouse clicks.** Element refs are robust against layout changes and resolution differences.

## Standard interaction flow

```bash
# 1. Discover and focus the target window
nib list-windows
nib focus-window "App Name"

# 2. Take an accessibility snapshot (interactive-only is faster)
nib snapshot --window "App Name" -i

# 3. Read the refs from the snapshot output, then interact
nib click-element @btn:Save --window "App Name"
```

## After UI changes (navigation, dialog open/close)

```bash
# Use diff to see what changed (faster than a full re-snapshot)
nib diff --window "App Name"

# Or take a fresh snapshot if the UI changed significantly
nib snapshot --window "App Name" -i
```

---

# Element refs

Snapshots assign stable refs to every interactive element. Refs encode the element type and label:

| Ref | Meaning |
|-----|---------|
| `@btn:Save` | Button labeled "Save" |
| `@txt:Username` | Text field labeled "Username" |
| `@chk:Remember` | Checkbox labeled "Remember" |
| `@btn~2:Submit` | 2nd button labeled "Submit" (disambiguated) |
| `@btn~1` | 1st untitled button |

**Type abbreviations:** btn=button, chk=checkbox, cmb=combobox, lnk=link, mnu=menuitem, rad=radiobutton, sld=slider, tab=tab, txt=textfield, txa=textarea, tri=treeitem, cel=cell, edt=editfield, lst=listitem, tgl=togglebutton, swt=switch, pbtn=popupbutton, mbtn=menubutton, mbar=menubaritem, dsc=disclosuretriangle, dock=dockitem, inc=incrementor, sbtn=splitbutton, clr=colorwell, spn=spinner, dat=dataitem, cmnu=checkmenuitem, rmnu=radiomenuitem

---

# Commands

## Window targeting

Most commands that operate on a window accept multiple ways to identify the target:

| Option | Description |
|--------|-------------|
| `--window <title>` | Match by window title (case-insensitive substring) |
| `--app <name>` | Filter by application name |
| `--pid <id>` | Filter by process ID |
| `--path <path>` | Filter by executable path |
| `--bundle-id <id>` | Filter by bundle identifier (macOS) |

Filters can be combined. If multiple windows match, you'll be asked to narrow your search.

## Window management

```
nib list-windows [--full] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib active-window [--full]
nib focus-window [title] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib window-region [title] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib resize-window [title] <width> <height>
nib move-window [title] <x> <y>
nib minimize-window [title]
nib restore-window [title]
nib window-elements [title] [--max N]
```

- Window titles are matched case-insensitively as substrings.
- `--full` on `list-windows` and `active-window` includes owner details (process ID, name, path, bundle ID) and window state.
- All window commands support window targeting options.

## Accessibility & element discovery

```
nib snapshot [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>] [-i] [--tree] [--compact] [--max N]
nib diff [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib find-element [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>] [--id <p>] [--type <t>] [--title <p>] [--role <r>] [--value <v>] [--class-name <p>] [--help-text <p>] [--selected-text <p>]
nib get-element <ref> --window <title> [--property title|value|role|type|region|states|all]
nib check-element <ref> --window <title> [--property visible|enabled|checked|focused|selected|expanded|readonly|required]
```

- Default snapshot returns a flat compact ref list (one line per element, minimal tokens).
- `-i` (interactive-only): omit non-interactive elements. Use this by default.
- `--tree`: return the full nested accessibility tree instead of a flat list.
- `--compact` (with `--tree`): collapse single-child unnamed wrapper nodes.
- `find-element` does a live search without needing a prior snapshot. Id/title/value/class-name/help-text/selected-text patterns are case-insensitive substring matches.
- `get-element` does a live query for current element properties.
- `check-element` returns a boolean check on element state.

## Element interaction

**Requires a snapshot first.** Uses refs from the most recent snapshot.

```
nib click-element <ref> [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib double-click-element <ref> [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib right-click-element <ref> [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib type-element <ref> <text...> [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib focus-element <ref> [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib hover-element <ref> [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
nib scroll-element <ref> <up|down|left|right> [amount] [--window <title>] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
```

`type-element` clicks the element to focus it, then types the text.

## Mouse (coordinate-based fallback)

```
nib mouse-position
nib mouse-move <x> <y>
nib mouse-move-smooth <x> <y>
nib click [--x X --y Y] [--button left|right|middle]
nib double-click [--x X --y Y] [--button left|right|middle]
nib right-click [--x X --y Y]
nib drag <fromX> <fromY> <toX> <toY>
nib scroll <up|down|left|right> [amount]
nib mouse-down [--button left|right|middle]
nib mouse-up [--button left|right|middle]
```

`--x` and `--y` must be provided together or both omitted (uses current position).

## Keyboard

```
nib type <text...>
nib press <keys...>
nib key-down <keys...>
nib key-up <keys...>
nib list-keys
```

`press` simulates pressing keys together (e.g., `nib press leftcmd c` for Cmd+C).
Key names are lowercase. Use `list-keys` to see all available names. Common aliases: enter, space, tab, esc, ctrl, alt, shift, cmd, left, right, up, down, backspace, delete.

## Screen

```
nib screen-size
nib screenshot [--file <path>] [--region x,y,w,h] [--format png|jpg]
nib screenshot-base64 [--region x,y,w,h] [--format png|jpg]
nib color-at <x> <y>
nib highlight <x> <y> <w> <h> [--duration ms] [--opacity 0-1]
nib find-text <text> [--regex] [--language eng,deu,...]
nib read [--region x,y,w,h] [--language eng,...] [--confidence 0-100]
```

- `screenshot` saves to file, `screenshot-base64` returns data URL.
- `find-text` uses OCR to locate text on screen and returns its region.
- `read` extracts text via OCR from the full screen or a region.

## Clipboard

```
nib clipboard-get
nib clipboard-set <text...>
```

## Wait & timing

```
nib wait <ms>
nib wait-for-window [title] [--timeout ms] [--interval ms] [--app <name>] [--pid <id>] [--path <path>] [--bundle-id <id>]
```

`wait-for-window` polls until a matching window appears (default timeout: 10s). Supports window targeting options.

## Batch execution

```
nib batch [file] [--stop-on-error]
```

Executes commands from a `.nib` script file or stdin (one command per line, `#` comments).

## System

```
nib version
nib status
```

---

# Common patterns

## Fill a form and submit

```bash
nib focus-window "My App"
nib snapshot --window "My App" -i
# Parse refs from output, then:
nib type-element @txt:Username "admin" --window "My App"
nib type-element @txt:Password "secret" --window "My App"
nib click-element @btn:Login --window "My App"
```

## Wait for a dialog, then interact

```bash
nib wait-for-window "Confirm" --timeout 5000
nib snapshot --window "Confirm" -i
nib click-element @btn:OK --window "Confirm"
```

## Verify state before acting

```bash
nib check-element @btn:Submit --window "App" --property enabled
# Parse the result field — only proceed if true
nib click-element @btn:Submit --window "App"
```

## Read text from a region

```bash
nib read --region 100,200,400,50
```

## Keyboard shortcuts

```bash
nib focus-window "Editor"
nib press leftcmd a          # Select all
nib press leftcmd c          # Copy
nib press leftcmd v          # Paste
```

## Navigate menus

```bash
nib snapshot --window "App" -i
nib click-element @mbar:File --window "App"
nib wait 300
nib snapshot --window "App" -i    # Re-snapshot to see menu items
nib click-element @mnu:Save --window "App"
```

## Track UI changes after an action

```bash
nib snapshot --window "App"              # baseline
nib click-element @btn:Submit --window "App"
nib wait 500
nib diff --window "App"                  # shows added/removed/changed elements
```

---

# Rules for reliability

1. **Always snapshot before using element refs.** Refs come from the last snapshot — stale refs will fail.
2. **Re-snapshot after UI transitions.** Opening a dialog, navigating pages, or expanding menus changes the element tree.
3. **Use `diff` for incremental updates.** After small changes (typing, toggling), `diff` is faster than a full snapshot.
4. **Use `-i` for snapshots.** Interactive-only mode is faster and produces less noise.
5. **Prefer element refs over coordinates.** Refs survive layout shifts, resolution changes, and minor UI updates.
6. **Use coordinates only as a last resort.** For elements without accessibility info (canvas, custom widgets), fall back to `find-text` + `click --x --y` or OCR-based approaches.
7. **Add short waits after actions that trigger animations or network requests.** Use `nib wait 300` to `nib wait 1000` depending on expected latency.
8. **Check element state before interacting** when the element may be disabled or hidden. Use `check-element` with `--property enabled` or `--property visible`.
9. **Handle errors gracefully.** If a ref is not found, re-snapshot. If a window is not found, use `wait-for-window`. If an action fails, read the error message.
10. **Window titles are case-insensitive substring matches.** Use a unique substring — you don't need the full title.
