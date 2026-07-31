# Favorites Panel

The extension adds a panel for accessing frequently used commands, files, directories, URLs, programs, snippets. The panel can be standalone or as part of the Explorer (in this case, you can drag the panel like any other to the desired location).

![Favorites Panel](preview/screenshot_0.png)

## Features

- Quick access to favorite commands
- Running multiple commands in sequence
- Quick access to your favorite files and folders
- Quick access to favorite URLs
- Fast launch of applications
- Setting icons for commands
- Separation setting for different workspaces

## Settings

Configure in VS Code settings (`settings.json`) under **`favoritesPanel.commands`**, or point to an external file via **`favoritesPanel.configPath`**.

For workspace-specific lists use **`favoritesPanel.commandsForWorkspace`** / **`favoritesPanel.configPathForWorkspace`** in the *workspace* settings (not User settings).

If nothing is configured, demo settings are used.

Load order (all sources are merged):

1. `favoritesPanel.commands`
2. `favoritesPanel.commandsForWorkspace`
3. `favoritesPanel.configPath`
4. `favoritesPanel.configPathForWorkspace`
5. `.vscode/favoritesPanel.json` in the project
6. `.favoritesPanel.json` in the project
7. `favoritesPanel.json` in the project

### favoritesPanel.commands

```json
"favoritesPanel.commands": [
    {
        "label": "README",
        "description": "- read me",
        "icon": "zap",
        "iconColor": "editorBracketHighlight.foreground5",
        "command": "openFile",
        "arguments": ["README.MD"]
    }
]
```

### favoritesPanel.configPath

External file path. Since 1.4.0 the file may be either a plain array of commands or the older wrapper object `{ "favoritesPanel.commands": [ ... ] }`.

```json
"favoritesPanel.configPath": "C:\\Projects\\favoritesPanel.json"
```

### favoritesPanel.explorerView

Moves the panel into the Explorer view so you can drag it elsewhere.

```json
"favoritesPanel.explorerView": true
```

Secondary Side Bar | Bottom Panel
:-------------------------:|:-------------------------:
![Favorites Panel](preview/screenshot_1_1.png) | ![Favorites Panel](preview/screenshot_1_2.png)

## Item fields

Required: **`label`**. Optional: **`description`**, **`icon`**, **`iconColor`**, **`command`** / **`arguments`**, **`sequence`**, or nested **`commands`** (group).

- Icons: [codicon listing](https://code.visualstudio.com/api/references/icons-in-labels#icon-listing)
- Colors: [theme color reference](https://code.visualstudio.com/docs/getstarted/theme-color-reference) (e.g. `editorBracketHighlight.foreground1`…`6`, `terminal.ansi*`)

Custom colors via `workbench.colorCustomizations`:

```json
"workbench.colorCustomizations": {
    "favoritesPanel.myColorGreen": "#006700"
}
```

Then `"iconColor": "favoritesPanel.myColorGreen"` on an item.

## Examples

### Transform selection

```json
{
    "label": "lowercase ➜ UPPER CASE",
    "icon": "debug-step-out",
    "command": "runCommand",
    "arguments": ["editor.action.transformToUppercase"]
}
```

### Open file (in project / external)

```json
{ "label": "README", "command": "openFile", "arguments": ["README.MD"] }
```

```json
{
  "label": "Hosts",
  "command": "openFile",
  "arguments": ["C:\\Windows\\System32\\drivers\\etc\\hosts", "external"]
}
```

### Run program / open folder (Windows)

```json
{ "label": "Chrome", "command": "run", "arguments": ["start chrome"] }
```

```json
{ "label": "Windows", "command": "run", "arguments": ["start explorer /n, C:\\Windows"] }
```

### Open URL

```json
{
  "label": "github.com",
  "command": "runCommand",
  "arguments": ["vscode.open", "https://github.com"]
}
```

### Reveal folder in Explorer

Workspace-relative paths are resolved like `openFile`. For an absolute path outside the workspace, pass `"external"`:

```json
{
  "label": "_work",
  "icon": "symbol-folder",
  "command": "runCommand",
  "arguments": ["revealInExplorer", "_work"]
}
```

### Find in files

```json
{
  "label": "Find in files",
  "command": "runCommand",
  "arguments": ["workbench.action.findInFiles", {"query": "SearchPattern", "triggerSearch": true}]
}
```

### Insert / replace text (`insertNewCode`)

Args: `[file, searchPattern, newCode, action]` where `action` is `before` (default), `replace`, or `replaceAll`.

```json
{
  "label": "Replace",
  "icon": "find-replace",
  "command": "insertNewCode",
  "arguments": ["ui/components/tableItem.ts", "<td className=\"col-date-time\">", "<div className=\"WOW\"></div>", "replace"]
}
```

### Sequence

```json
{
    "label": "Sequence",
    "icon": "console",
    "sequence": [
        { "command": "runCommand", "arguments": ["workbench.action.terminal.new"] },
        { "command": "runCommand", "arguments": ["workbench.action.terminal.focus"] },
        {
            "command": "runCommand",
            "arguments": ["workbench.action.terminal.renameWithArg", { "name": "New Terminal" }]
        },
        {
            "command": "runCommand",
            "arguments": [
                "workbench.action.terminal.sendSequence",
                { "text": "node -v\nnpm -v\ngit --version\n" }
            ]
        }
    ]
}
```

### Starter snippet

Copy into `settings.json`:

```json
"favoritesPanel.commands": [
    {
        "label": "README",
        "description": " - Important!",
        "command": "openFile",
        "arguments": ["README.MD"]
    },
    {
        "label": "EDIT",
        "commands": [
            {
                "label": "lowercase ➜ UPPER CASE",
                "icon": "debug-step-out",
                "command": "runCommand",
                "arguments": ["editor.action.transformToUppercase"]
            }
        ]
    },
    {
        "label": "github.com",
        "icon": "link-external",
        "command": "runCommand",
        "arguments": ["vscode.open", "https://github.com"]
    },
    {
        "label": "ZOOM",
        "commands": [
            {
                "label": "Zoom In",
                "icon": "zoom-in",
                "command": "runCommand",
                "arguments": ["editor.action.fontZoomIn"]
            }
        ]
    }
]
```

## Release Notes

### 1.4.5 | 2026/06/07
- Fixed empty lines being inserted in settings.json when opening settings from the panel toolbar.

More information in the [changelog](CHANGELOG.md).

---

## Original Extension

This extension is a fork of the original [Favorites Panel](https://github.com/sabitovvt/vscode-favorites-panel) extension developed by [Vladimir Sabitov](https://github.com/sabitovvt). All credits for the original idea and core implementation go to the original author.
