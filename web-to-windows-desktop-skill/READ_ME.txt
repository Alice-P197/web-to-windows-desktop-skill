================================================================================
  web-to-windows-desktop — Skill Documentation Package
================================================================================

  Version:  1.0.0
  Date:     2026-08-06
  Author:   Robin
  Language: English (SKILL.md) / Chinese (web-to-windows-desktop.md)

================================================================================
  FILE MANIFEST
================================================================================

  SKILL.md
    The skill definition document. Written in English. Contains the complete
    execution workflow that an AI agent follows when this skill is triggered:
    parameter parsing, project scaffolding, file generation, dependency
    installation, build commands, output verification, and error handling.
    This is the file to import into the AI skill system.

  web-to-windows-desktop.md
    The human-readable explanation document. Written in Chinese. Explains
    the skill's purpose, technical architecture, supported input modes,
    configurable parameters, comparison of the two output artifacts,
    security design, and frequently asked questions. Suitable for
    developers and end users who want to understand how the skill works.

  READ_ME.txt
    This file. Contains the package manifest, changelog, and usage
    guidance for the end user.

================================================================================
  CHANGELOG
================================================================================

  v1.0.0 (2026-08-06)
    - Initial release
    - Support for both web URL and local HTML file inputs
    - Dual output: portable executable + NSIS installer
    - Electron 33 + electron-builder 25
    - 64-bit Windows target only
    - Auto icon fallback
    - Single-instance lock
    - Context isolation and sandbox enabled by default

================================================================================
  USER GUIDANCE
================================================================================

  How to use this skill:

  1. Import SKILL.md into your AI skill system.
  2. Provide the AI with a URL or local HTML file path.
     Example: "Package https://example.com into a Windows desktop app"
     Example: "Turn /path/to/index.html into an exe"
  3. The AI will automatically scaffold the project, install dependencies,
     build both artifacts, and deliver the output files.
  4. You will receive two files:
       - Portable executable: runs immediately, no installation needed
       - NSIS installer: standard Windows setup wizard

  Prerequisites before using the skill:

    - Node.js >= 18.0.0
    - npm >= 9.0.0
    - At least 2 GB of free disk space
    - Windows 10/11 (or macOS/Linux with Wine installed)

  Output location:

    Both executable files will be placed in the workspace directory
    (/workspace) for easy access.

================================================================================
  NOTES
================================================================================

  - The packaged application size is approximately 80-120 MB because
    Electron bundles a full Chromium browser engine and Node.js runtime.
  - Only trusted web content should be packaged, as the resulting
    application has the same system privileges as any Electron app.
  - Nativefier (a similar tool) has been unmaintained since September 2023.
    This skill uses the latest Electron and electron-builder versions directly.

================================================================================
  END OF FILE
================================================================================