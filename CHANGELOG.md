# Changelog

All notable changes to Mythos Controller. Versions follow `MAJOR.MINOR.PATCH`; each release is a
git tag `vX.Y.Z` on `main`, and older releases stay available so you can go back
(see `docs/RELEASE.md`).

## 1.1.0 - 2026-10-12

First stable release distributed as an installer and a ZIP.

### Added
- **Custom Code per key** (Python or PowerShell): checked without running, activated explicitly,
  run in a separate windowless process with a timeout; imported code is deactivated.
- **Copy a key's setting by drag and drop**: press a key that has a setting, drag it onto another
  key and release. The target gets a complete copy (action, shortcut, macro steps, texts, Custom
  Code, colour, tag, options) and keeps its own key; nothing is executed; an invalid drop changes nothing.
- **Open Program launchers**: reuse the running copy of a program instead of starting another;
  installed-application picker (keeps shortcut arguments).
- Key search with highlighting, a **Test** button in the editor, an explicit ON/OFF badge.
- Diagnostics (Advanced tab): `ambiguous` and `own output` counters and the last key decisions
  with the reason a key was swallowed (`run`, `unmapped`, `repeat`, `release`, ...).
- Arabic printing fixes, accessibility fixes, the Mythos Design System (tokens and self-hosted fonts).
- Windows installer (Inno Setup, per-user, no administrator rights) in addition to the ZIP,
  with SHA-256 checksums.

### Changed
- **Keyboard hook protocol v2**: one shared memory layout for the 32-bit and 64-bit builds and
  one question per keystroke (version 1 asked twice and ignored the key list). The application
  refuses any hook DLL that does not report protocol 2 and the expected layout. The 32-bit hook
  DLL and its host are no longer shipped (Windows calls the 64-bit hook back for 32-bit programs).
- The packaged application loads the hook DLL from a private, content-named copy in
  `%LOCALAPPDATA%\MythosKeyboardController\native`, so updates and uninstalls are never blocked by
  Windows keeping the DLL mapped in other programs.

### Fixed
- **Logs no longer contain login links, one-time codes or tokens**: every URL is reduced to
  `scheme://host` in the log (the browser still gets the full address).
- A delayed clipboard restore after a paste no longer overwrites something copied afterwards, and
  two quick pastes restore the real original clipboard.
- A Keyboard 2 key whose release was lost (focus moved to a window the hook cannot enter) is no
  longer treated as an endless auto-repeat; it works again on the next press.
- A Keyboard 2 key that has no setting is logged instead of vanishing silently.

### Security and privacy
- The release is built by GitHub Actions from the tagged commit; the published files are those
  tested by the workflow. `SHA256SUMS.txt` lists their checksums.
- Settings, backups and logs stay in `%APPDATA%\MythosKeyboardController`, outside the program
  folder; installing, updating and uninstalling never delete them.
- The release files contain no configuration, logs or credentials.

### Known limitations
- The executable and the hook DLL are **not code-signed**: Windows SmartScreen may warn on first
  run ("More info" -> "Run anyway"), and some antivirus tools are cautious about any program that
  installs a keyboard hook.
- The keyboard hook does not reach **elevated (administrator) windows** or Microsoft Store (UWP)
  apps: in those windows a Keyboard 2 key types normally and its action does not run.
- Requires the **Microsoft Edge WebView2 Runtime** (included with Windows 11 and with Microsoft Edge
  on Windows 10).
- Custom Code in **Python** needs Python (or the `py` launcher) installed; PowerShell code works as is.
- With Keyboard 2 ON, every key of Keyboard 2 is swallowed; keys without a setting do nothing.
- Going back to 1.0.0 is not supported if you use Custom Code keys (1.0.0 rejects that action): back up
  `%APPDATA%\MythosKeyboardController` first (see `docs/RELEASE.md`).
- Tested on Windows 10 Pro 22H2 (build 19045). Windows 11 is expected to work but was not tested.

## 1.0.0

Initial version (see `docs/PROJECT_STATUS.md`).
