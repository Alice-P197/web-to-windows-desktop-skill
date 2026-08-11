# SKILL: web-to-windows-desktop

## Skill Name

`web-to-windows-desktop`

## Version

1.0.0

## Description

Package any web URL or local HTML file into a native 64-bit Windows desktop application. The skill produces two deliverables in a single build:

- **Portable executable** — a standalone `.exe` that runs immediately without installation
- **NSIS installer** — a standard Windows setup wizard that creates desktop shortcuts, Start Menu entries, and supports uninstallation

Under the hood, the skill scaffolds an Electron project, configures electron-builder with dual targets (portable + NSIS), builds the artifacts, and delivers both to the user.

## Trigger Conditions

Activate this skill when the user request contains any of the following intent signals:

- "package a webpage as a desktop app"
- "turn a website into an exe"
- "create a Windows desktop app from HTML"
- "generate a Windows installer from a URL"
- "make a portable exe from a web page"
- "wrap a local HTML file as a Windows app"
- Any request to convert a URL or `.html` file into a `.exe` program

## Input Parameters

Extract the following from the user's request. Use defaults for missing optional parameters, or ask the user when the parameter is critical.

| Parameter | Required | Default | Description |
|-----------|----------|---------|-------------|
| `url` | Yes (if no local file) | — | Target web page URL, must include protocol (`https://`) |
| `htmlPath` | Yes (if no URL) | — | Absolute path to a local HTML file |
| `name` | No | Auto-extracted from page `<title>` | Application display name, used for window title, shortcuts, and install directory |
| `appId` | No | `com.${sanitizedName}.app` | Unique application identifier in reverse-domain format |
| `icon` | No | Electron default icon | Path to a `.png` or `.ico` icon file (recommended: 256x256) |
| `width` | No | 1280 | Default window width in pixels |
| `height` | No | 800 | Default window height in pixels |
| `minWidth` | No | 800 | Minimum window width |
| `minHeight` | No | 600 | Minimum window height |
| `singleInstance` | No | true | Whether only one instance of the app may run at a time |
| `outputDir` | No | `./dist` | Build output directory |

## Workflow

### Step 1: Parse User Intent

1. Extract the target URL or local HTML file path from the user's request.
2. If the URL lacks a protocol, prepend `https://`.
3. If the user provides a local file path, verify the file exists and read its content. Copy it into the project directory as an embedded asset.
4. If the user does not provide an application name, defer extraction until the Electron window loads the page title.
5. Collect all optional parameters (icon, window dimensions, etc.).

### Step 2: Create Project Scaffold

Create the following directory structure under a temporary working directory:

```
project/
├── package.json
├── main.js
├── preload.js
├── electron-builder.yml
└── assets/
    └── icon.png          (optional)
```

### Step 3: Generate Core Files

#### package.json

```json
{
  "name": "${sanitizedName}",
  "version": "1.0.0",
  "description": "Desktop wrapper for ${url}",
  "main": "main.js",
  "author": "",
  "license": "MIT",
  "scripts": {
    "start": "electron .",
    "build": "electron-builder --win portable nsis --x64"
  },
  "devDependencies": {
    "electron": "^33.0.0",
    "electron-builder": "^25.0.0"
  }
}
```

#### main.js

The main process script must handle:

- **URL mode**: Load the target URL via `mainWindow.loadURL(url)`.
- **Local file mode**: Load the embedded HTML file via `mainWindow.loadFile(path)`.
- **External link handling**: Intercept `will-navigate` and `new-window` events to open external links in the system default browser via `shell.openExternal()`.
- **Single instance lock**: Use `app.requestSingleInstanceLock()` to prevent duplicate instances.
- **Window lifecycle**: Restore minimized windows on second-instance events; quit when all windows are closed.

```javascript
const { app, BrowserWindow, shell } = require('electron');
const path = require('path');

const TARGET_URL = '${url}';           // or null for local file mode
const HTML_PATH = '${htmlPath}';       // or null for URL mode
const WIN_WIDTH = ${width};
const WIN_HEIGHT = ${height};
const SINGLE_INSTANCE = ${singleInstance};

let mainWindow = null;

function createWindow() {
    mainWindow = new BrowserWindow({
        width: WIN_WIDTH,
        height: WIN_HEIGHT,
        minWidth: ${minWidth},
        minHeight: ${minHeight},
        icon: path.join(__dirname, 'assets', 'icon.png'),
        webPreferences: {
            preload: path.join(__dirname, 'preload.js'),
            contextIsolation: true,
            nodeIntegration: false,
            sandbox: true
        }
    });

    mainWindow.webContents.setWindowOpenHandler(({ url }) => {
        if (url && (url.startsWith('http://') || url.startsWith('https://'))) {
            shell.openExternal(url);
        }
        return { action: 'deny' };
    });

    if (TARGET_URL) {
        mainWindow.loadURL(TARGET_URL);
    } else {
        mainWindow.loadFile(HTML_PATH);
    }

    mainWindow.on('closed', () => { mainWindow = null; });
}

const gotLock = app.requestSingleInstanceLock();
if (SINGLE_INSTANCE && !gotLock) {
    app.quit();
} else {
    app.on('second-instance', () => {
        if (mainWindow) {
            if (mainWindow.isMinimized()) mainWindow.restore();
            mainWindow.focus();
        }
    });
}

app.whenReady().then(createWindow);
app.on('window-all-closed', () => app.quit());
app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) createWindow();
});
```

#### preload.js

Expose a minimal, safe API to the renderer process:

```javascript
const { contextBridge } = require('electron');

contextBridge.exposeInMainWorld('desktopApp', {
    platform: process.platform,
    version: '1.0.0'
});
```

#### electron-builder.yml

```yaml
appId: ${appId}
productName: ${name}

directories:
  output: ${outputDir}
  buildResources: assets

win:
  target:
    - target: portable
      arch: [x64]
    - target: nsis
      arch: [x64]
  icon: assets/icon.png

nsis:
  oneClick: false
  perMachine: false
  allowToChangeInstallationDirectory: true
  createDesktopShortcut: true
  createStartMenuShortcut: true
  shortcutName: ${name}
  uninstallDisplayName: "Uninstall ${name}"

portable:
  artifactName: "${name}-portable.exe"

files:
  - "main.js"
  - "preload.js"
  - "assets/**/*"
  - "node_modules/**/*"
  - "!node_modules/**/*.md"
  - "!node_modules/**/*.map"
  - "!node_modules/**/test/**"
  - "!node_modules/**/docs/**"

asar: true
compression: maximum
```

### Step 4: Install Dependencies

```bash
cd project/
npm install
```

If the npm registry is slow or unreachable, switch to a mirror:

```bash
npm config set registry https://registry.npmmirror.com
```

### Step 5: Build

```bash
npm run build
```

This executes `electron-builder --win portable nsis --x64`, producing both targets in a single run.

### Step 6: Verify Outputs

After the build completes, confirm the following artifacts exist:

| Artifact | Expected Path | Description |
|----------|---------------|-------------|
| Portable executable | `${outputDir}/${name}-portable.exe` | Single-file executable, double-click to run, no installation required |
| NSIS installer | `${outputDir}/${name} Setup 1.0.0.exe` | Standard Windows installer with setup wizard, desktop shortcut, Start Menu entry, and uninstall support |
| Unpacked directory | `${outputDir}/win-unpacked/` | Full unpacked application directory (for debugging) |

### Step 7: Deliver to User

1. Copy both executable files to the `/workspace` directory.
2. Report file paths and sizes to the user.
3. Provide usage instructions:
   - **Portable**: Double-click `${name}-portable.exe` to run immediately.
   - **Installer**: Double-click the setup exe, follow the wizard, and the app will be available from the desktop and Start Menu.

## Platform Compatibility

| Build Platform | Can Build Windows x64? | Requirement |
|----------------|------------------------|-------------|
| Windows 10/11 | Yes | No extra dependencies |
| macOS | Yes | Wine must be installed (`brew install --cask wine-stable`) |
| Linux | Yes | Wine must be installed (`sudo apt install wine64`) |

## Error Handling

| Error Scenario | Handling Strategy |
|----------------|-------------------|
| Node.js version < 18 | Prompt user to upgrade Node.js |
| Target URL unreachable | Warn user but continue building (the app may still be useful offline) |
| npm install fails | Retry with mirror registry; check network connectivity |
| electron-builder fails | Check disk space (needs ~2 GB); verify platform is Windows or Wine is installed |
| Icon file missing | Use Electron's default icon; do not block the build |
| Local HTML file not found | Report the error and ask the user to verify the path |

## Security Considerations

- The packaged application runs with the same system privileges as Electron/Chromium. Only package content from trusted sources.
- `contextIsolation: true` and `sandbox: true` are enabled by default to restrict renderer process capabilities.
- External links open in the system default browser, not within the Electron shell, to prevent phishing.
- Do not embed API keys, tokens, or credentials in the packaged application.

## Output Format

After a successful build, report to the user:

1. Full paths to both output files
2. File sizes of the portable and installer executables
3. Usage instructions for each artifact