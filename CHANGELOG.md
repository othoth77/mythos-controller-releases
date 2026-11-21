# Changelog

All notable changes to Mythos Controller. Versions follow `MAJOR.MINOR.PATCH`; each release is a
git tag `vX.Y.Z` on `main`, and older releases stay available so you can go back
(see `docs/RELEASE.md`).

## 1.6.0 - 2026-11-21

### MYTHOS INTERRUPT (`docs/INTERRUPT.md`)
- New default rule: while an action, a sequence (nested ones included), a mouse playback or a test
  is executing, manual input stops it at once.
  - Manual input is: the real mouse moved, a click, the wheel, or a key pressed on any keyboard.
  - A remaining delay is not waited for, the steps that were left never run, and nothing resumes.
  - Keys and mouse buttons still held by the execution are released.
- Another Mythos action started meanwhile stops the running one, then runs. The two never overlap.
  Before, the new key was ignored while a sequence ran.
- Only real hardware counts. The keys, mouse moves and clicks Mythos Controller generates never
  interrupt the execution that sends them. No hook is added and nothing is blocked.
- Not counted: key and button releases, the auto-repeat of a held key, a mouse at rest, mouse
  movement in the first instants of an execution, and anything you do while recording.
- An interruption is reported as « interrompu » with its cause, not as an error.
- Settings → **MYTHOS INTERRUPT**: on by default, also for existing configurations. Each cause can
  be turned off (mouse movement, click or wheel, keyboard, another action).
- Scripts launched by a custom-code key are not stopped by default; an option turns that on. A code
  step inside a sequence always stops with the sequence.
- One central service (`src/mkc/core/interrupt.py`) tracks everything that executes; sequences,
  playbacks, key actions and tests all use it.

### Changed
- The recorder and MYTHOS INTERRUPT share one mouse Raw Input registration. It is active only while
  a recording or an execution runs.

## 1.5.0 - 2026-11-21

### MYTHOS PROFILE SWITCHER: floating toolbar (`docs/PROFILE_SWITCHER.md`)
- The switcher is now a small borderless toolbar of mini profile buttons, just above the Windows
  taskbar next to the clock. It replaces the 1.4 notification-area icons: nothing in the taskbar,
  the notification area or the hidden-icons menu (^), no menu, no popup.
- One click on a button activates the profile at once. The toolbar stays on screen, also when
  Mythos Controller is minimized, and never takes the keyboard focus. The active profile is filled
  with its colour and outlined; a tooltip gives the full name.
- Place: by default next to the clock, on the bottom-right corner (top-right with a top taskbar,
  bottom-left with a left one).
  - Drag the grip (⋮⋮) to move it. It is kept inside the screen's work area, so it never covers the
    taskbar, the clock or the system icons, also with an auto-hidden taskbar.
  - The screen and the place are saved and restored after a restart.
- Monitor 1 / Monitor 2: choose the screen in Settings → Profile Switcher, or drag the toolbar onto
  it. If that screen is disconnected, the toolbar goes to the main screen.
  - « Réinitialiser la position » brings it back next to the clock.
  - It follows each screen's scale (100–200 %).
- Still optional and off by default. Existing settings (on/off, label, visible profiles) are kept;
  « Afficher dans la zone de notification » becomes « Afficher la barre ». New settings
  `profile_switcher_monitor` and `profile_switcher_offset` are empty by default, meaning the default
  place.

## 1.4.1 - 2026-11-21

### Fixed
- **Image targets (MYTHOS MOUSE ENGINE / TARGET MEMORY)**: an image that appears twice on screen is
  now always refused as ambiguous, including at its expected place (it used to click one of them).
  An image present only once is found at every position: the search no longer misses copies that
  do not fall on the coarse search grid (it now tries four grid offsets and several candidates per
  offset, then refines each one at full resolution).
- **Profile Switcher icons disappearing**: the icon window ignores close requests that do not come
  from Mythos Controller. If its thread ends anyway, the switcher notices within a second and
  recreates the icons. It logs the reason as a warning and does this at most 5 times per 10 minutes;
  Settings then shows the number of restarts.
- **Window class reuse**: the Profile Switcher and the mouse recorder each register their hidden
  window class once per process, with one dispatcher. Before, a second start could call the
  procedure of a stopped instance. This made the icons stop responding after a disable/enable, and
  the recorder could crash with an access violation.

## 1.4.0 - 2026-11-20

### MYTHOS PROFILE SWITCHER (`docs/PROFILE_SWITCHER.md`)
- Optional (off by default): one compact icon per profile in the Windows notification area, near the
  clock. One click activates the profile at once, without opening the window and without
  confirmation; the active profile is filled with its colour and outlined; the tooltip gives the
  full name.
- Settings → **Profile Switcher**: enable / disable, show / hide in the notification area, label
  (profile number, short name or the start of the name), visible profiles (all; only the chosen
  ones — new profiles hidden; all except the chosen ones — new profiles shown). Kept across restarts
  of Mythos Controller and of Windows.
- Always in step with the main window: active profile, profile creation, renaming, deletion,
  colours and settings; a click on a profile just deleted or hidden does nothing.
- Safe switching: a profile chosen from these icons never interrupts a running sequence, mouse
  playback or recording (they never depend on the active profile); the new profile applies to the
  next key presses.
- One hidden window and one thread, no keyboard hook; icons added again when Explorer restarts;
  removed when Mythos Controller exits; if the notification area is unavailable the application
  continues and Settings says so.

### Known limitations
- Windows decides where new notification icons go: on Windows 10/11 they may land in « Afficher les
  icônes cachées » (^) — drag them onto the taskbar once to keep them next to the clock.
- A notification icon is 16–32 pixels: « Nom » shows the first letters, the full name is in the
  tooltip.

## 1.3.0 - 2026-11-20

Everything below up to 1.2.0 is part of 1.3.0 (1.2.0 was never published: users of 1.1.0 get all of it).

### MYTHOS MOUSE ENGINE (`docs/MOUSE_ENGINE.md`)
- **Recording** of the mouse (movement, left / right / middle / X buttons, double-click, button
  down/up, drag and drop, vertical and horizontal wheel) and of the keys of the normal keyboards
  (key down/up, combinations), with high-resolution timestamps; start (3 s countdown), pause,
  resume, stop, cancel. Mouse input is observed through passive Raw Input, keys through a copy of the
  records the keyboard backend already receives: no new hook, nothing blocked or delayed. Keyboard 2
  and every injected key are never recorded; clicks on Mythos Controller itself are left out.
- A bounded queue and a worker thread (movements thinned, never clicks or keys), a journal file on
  disk for long recordings, watchdog status (queue, dropped movements, late worker).
- **Editor**: ordered timeline, add / delete / duplicate / reorder, coordinates, delays, durations,
  buttons, keys, text; manual mouse and keyboard actions; « Clic à la position de la souris »;
  playback speed, repetitions, « jusqu'à l'arrêt », pause between repetitions; Test before saving.
- **Playback**: once, n times or until stopped, pause / resume / stop; every key and mouse button
  still held is released when playback ends, is stopped or fails. Emergency stop: « ■ Arrêter », the
  « Arrêter la séquence » key, a physical Échap, or the real mouse thrown into the top-left corner
  (fail-safe, can be switched off).
- **Coordinates**: absolute (virtual desktop), relative to a monitor (scaled to its resolution) or
  to a window's client area (follows moves, optionally resizes); multi-monitor, per-monitor DPI aware
  (physical pixels on every scaling); conversion between frames in the editor.
- **Smart Mouse**: Windows UI Automation (through .NET), image search (built-in, Pillow; OpenCV used
  automatically when installed), waiting for an element, checking that it exists, verifying the
  target window; ambiguous or weak matches are refused, never guessed.
- New key actions « Enregistrement souris » and « Action souris » (click, double-click, right or
  middle click, move, scroll, drag) — the latter at the key's execution target.

### MYTHOS TARGET MEMORY (`docs/TARGET_MEMORY.md`)
- Targets stored once with stable ids, versions and learning method: **Last Learned Position**,
  **Specific Window** (follows the window), **Smart Target** (UI Automation id / name / type, image,
  relative fallback only when allowed); plus **Écran 1**, **Écran 2** (configurable monitors) and
  **Écran actif**. Learn (countdown, review, name, save), test (the mouse shows where, no click),
  re-learn, rename, duplicate, options, delete (usages shown, confirmation).
- **« Cible d'exécution » on every key** of every profile, page and keyboard, whatever its action:
  Default (unchanged behaviour), one active target and several saved ones, click-to-focus, on/off.
  One central service resolves, validates and applies targets: input goes to the target window /
  point, programs and websites open on the target screen, custom code receives `MC_TARGET_*`,
  recordings play relative to the target. **A target that cannot be validated is never executed.**

### MYTHOS SEQUENCER integration
- New steps: mouse (click, double-click, right/middle, move, down/up, scroll, drag — at a target, at
  x, y or at the mouse), keyboard (key, down/up, shortcut, text), recording (× n), wait for a target
  (timeout; then stop, continue or skip the next step: a condition), and repeat of a sub-sequence
  (loop). Button steps run with their execution target. Strict order, long actions awaited, one stop
  reaching every level and the recording being played, errors naming the sequence path and the step.

### Action Hub
- `.mcsequence` bundles carry recordings (positions only relative to a window or a monitor, window
  titles removed, typed text through parameters) and targets reduced to non-private identity
  (program, class, UI Automation id/type): imported targets must be taught on the new PC first;
  absolute positions become targets to teach. Nothing imported runs by itself.
- The Action Hub library is part of the main project (`hub-library/`, `tools/hub_sync.py`).

### Compatibility
- Configuration version 5 (versioned migration from 4: existing data unchanged, the previous file
  kept as `config.pre-v5.json`); keys without a target are stored exactly as before.
- Keyboard 1 untouched; Keyboard 2 keys other than Stop / profile / page are ignored while a sequence
  or a playback runs (never queued).

### Known limitations
- Windows does not let a normal program send input to, or read UI Automation of, windows running as
  administrator (UAC) or to the secure desktop; such targets are reported « non valide ».
- In a Remote Desktop session the mouse follows the remote session; recording a minimised RDP window
  captures nothing.
- The in-app updater still needs the release signing key (UPD-1): update with the installer.

### MYTHOS SEQUENCER v2 and NumEnter v6 (PR #10, #11, #15)

### Added
- **MYTHOS SEQUENCER** (`docs/SEQUENCER.md`): sequences are independent objects stored once
  (`config.sequences`: id, name, description, ordered steps, version, creation and change dates)
  and used by reference from keys (« Sequential Actions (Mythos Sequencer) », `run_sequence`) and
  from other sequences. Steps: a **button** of any page of **any profile** (run in its own context,
  the active profile never changes), another **sequence** (any depth, reusable everywhere) or a
  **delay**. No artificial limit on steps or levels (bounds: 64 levels, 10 000 actions and 2 h of
  delays per press, a 3 h watchdog).
- **⛓ Sequencer** section: list with content, usages and state; create, change, rename, duplicate,
  delete (usages shown, explicit confirmation); editor with button picker (profiles → pages →
  buttons with name, physical key, profile, page, action type), sequence and delay steps, ↑ ↓ ✕,
  dependencies (« Utilise / Utilisée par »), full preview of the tree, Test of the unsaved draft,
  Stop.
- Execution: everything one press would run is checked first (missing, disabled, non-activated
  code, cycle — also indirect and through keys —, bounds); strict order on one thread; custom code
  waited for; the first failing step stops the run and is named with its level; one stop reaches
  every level; never runs while being edited.
- **Action Hub**: a whole sequence is shared with its dependencies as a `.mcsequence` bundle
  (integrity per component, permissions, parameters instead of personal values, code never active);
  import = review then explicit install into a new page; nothing runs.
- NumEnter v6 (PR #11) and the Sequencer work together: a custom-code step of a sequence gets the
  key press context (window in front, last terminal).

### Changed
- The first Sequencer format (`run_keys`, PR #10) is converted on load to a stored sequence; the
  key keeps its name, colour and page.

### In-app updates and BRIDGE interface (PR #14)

### Added
- **Settings → Mises à jour** (French / Arabic): installed and latest version, publication date,
  release notes, included components, availability, download progress, signature and SHA-256
  verification, result, last successful update, update history; « Vérifier les mises à jour »,
  « Installer la mise à jour », « Voir les détails », « Historique des mises à jour », « Revenir à la
  version précédente ». Nothing is checked or installed without a button press (`docs/UPDATES.md`).
- Update Manager (`src/mkc/update/`):
  - one official source (`othoth77/mythos-controller-releases`) and a strict signed release
    manifest `mythos-controller-release/1` (`docs/RELEASE_MANIFEST.md`);
  - Ed25519 signature verification (pure Python, RFC 8032 vectors, cross-checked with OpenSSL);
  - size and SHA-256 checks after the download and again right before execution;
  - downgrade refused; one operation at a time;
  - no elevation: a per-user installation in a writable folder only;
  - the previous version is snapshotted and restored automatically when the installer or the first
    start of the new version fails (`docs/UPDATE_ROLLBACK.md`, `docs/UPDATE_SECURITY.md`).
- BRIDGE background service integration interface (`src/mkc/bridge_service/`,
  `docs/BRIDGE_SERVICE.md`): contract `bridge-windows-service/1` (least-privilege account,
  loopback-only health, no shell arguments), lifecycle with consent, independent updates with
  rollback, and uninstall that keeps the configuration. It is tested with a simulated service manager.
  **Not connected to Windows:** BRIDGE publishes no Windows service yet, so the status reads
  « Non disponible » with the missing deliverables.
- Release tools: `tools/make_release_manifest.py` and `tools/sign_release_manifest.py` (offline
  signing; the key is refused inside the repository).
- Release workflow: installer smoke test of the packaged update helper. An unsigned update is refused
  before the installer runs; a rollback restores the installation exactly.
- CI: syntax check of `hub.js` and `updates.js`.

### Not enabled yet (owner decisions, `docs/UPDATE_SECURITY.md`)
- No release signing key exists, so this build trusts none. Every update check ends with
  « Authenticité invérifiable » and nothing can be installed from the application until the owner
  generates the key offline and adds its public half (UPD-1).
- Releases up to 1.1.0 publish no manifest. Install them manually (UPD-2).

### Mythos Action Hub (PR #4–#9)

### Fixed (branch `fix/numenter-reliability`)
- **Key code that targets « the terminal » chose the wrong window intermittently.** Custom code
  now receives the window that was in front **when the key was pressed** and the last terminal
  window the user brought to the front (`MKC_PRESS_FOREGROUND_HWND`, `MKC_LAST_TERMINAL_HWND`,
  `MKC_LAST_TERMINAL_AGE_MS`; a foreground event hook, no polling). Before, the code looked for it
  itself 1–3 s later (PowerShell start-up), when Remote Desktop, a popup or another program could
  already be in front, and guessed « the last terminal » from the window order.
- A key pressed again while its code is still running was ignored without a trace; it is now
  logged (WARNING) so a press that « did nothing » can be explained.
- **NumEnter v6** (`tools/keys/numenter/`): the « copier » key script, now maintained and tested in
  the repository. Target from the press context (never a guess between several terminals), Windows
  Terminal tab by title, SSH tunnels skipped; waits until the command is over (prompt at the cursor,
  no program still attached to a local shell, screen quiet), consistent read of the buffer; prompts
  found after `cd`, a venv or with text typed after the prompt; history read without blocking
  PSReadLine; non-blocking Win32 clipboard with bounded retries (the OLE clipboard waited as long
  as another program held it), exact read-back, previous text restored and reported; refusal popup
  that never keeps the run alive; precise error codes and timings in `numenter.log` (never the
  copied text). `tools/keys/numenter/apply.py` installs it into a key.

### Added
- **Mythos Action Hub** (`docs/ACTION_HUB.md`): the « ⬡ Hub » button opens Discover, Incoming, My
  nominations, Library, Installed, Updates and History. Actions are shared through a public
  GitHub-backed library (read without an account or token) or a shared folder; proposing one opens
  a prefilled GitHub issue in the browser. Reviewing and installing never run anything and never
  change a key; installed code is deactivated; giving an action to a key asks before replacing a
  setting. Local customisation keeps the original; updates show the differences and are accepted
  or refused by the user; the previous version is kept for rollback. Maintainer tool:
  `tools/hub_publish.py`.
- Settings → Hub: approval mode (show and let me decide / only on request / off), library source.
- **⬡ Nominate to Action Hub** in the key editor (simple and advanced) for a configured key: a
  read-only preview of the shared action, then name, description, category and tags, and the
  existing nomination workflow (check, export, prefilled GitHub issue). Only the key's action is
  shared; user folders in program paths become `%USERPROFILE%` / `%APPDATA%` / `%LOCALAPPDATA%`,
  and a personal folder left in a text, arguments or code blocks the nomination. A key that
  received a Hub action shows « Mythos Action Hub : id version ». « Mes propositions » shows the
  key a nomination came from (kept on this PC only).

- **Simple « ⬡ Proposer à Action Hub »** (replaces the long form): name (prefilled),
  optional description (written automatically when empty), category (preselected), a live
  preview, then « Confirmer la proposition ». Author, id, version, licence, summary,
  compatibility and technical fields are generated; the advanced options (and « Vérifier ») are
  optional. **Fixes « manifest.author.name: required »**: the author is taken from the advanced
  field, Settings → Hub, the GitHub account, the Git user name, else « Mythos Controller User »;
  the GitHub account is optional and an invalid one never blocks.
- **Parameters for PC-specific values** (package format 2): user folders, `user@server`, IP
  addresses, e-mail addresses and the Windows account name found in texts, arguments or code are
  replaced by `{{mc:…}}` parameters (listed in the preview, each can be kept). Recipients fill
  their own values in « Installées » → « Paramètres de ce PC » (kept on their PC only) before
  giving the action to a key. Packages without parameters stay format 1, unchanged.
- Long actions: the package travels compressed inside the GitHub issue, or the full issue text is
  copied to the clipboard to paste into the opened page; attaching the file is only needed for a
  package larger than a GitHub issue.

### Fixed
- Secret scan: `API_TOKEN = '…'`, `db_password_prod = '…'` and similar prefixed names are now
  refused; a marker named « token » that does not look like one (`TOKEN = '[MKC:key]'`) no longer
  blocks a nomination.

### Configuration
- New optional `settings.hub_*` keys with safe defaults. No configuration version change, no
  migration: older versions ignore them. Hub data lives in `%APPDATA%\MythosKeyboardController\hub\`.

### Known limitations
- The public library repository does not exist yet; packages are not signed; custom code is not
  sandboxed (it is deactivated until the user activates it).

## 1.2.0 - 2026-10-27

Standard profile, Keyboard 2 profile shortcuts, action sequences, copy between profiles, and the
fixes of the 2026-10-27 audit (details and evidence in `docs/DEVLOG.md`).

### Added
- **Standard profile** (always first, ⌨): the normal keyboard. While it is active Mythos Controller
  runs nothing; any other profile turns Keyboard 2 into the macro keyboard. It replaces the ON/OFF
  switch (a key mapped to *Toggle macros* switches Standard ↔ last profile). The first start adds it
  and keeps every profile, page and key; a configuration saved OFF starts in Standard. The file as it
  was before that first start is kept as `config.pre-1.2.0.json` next to `config.json`.
- **Profile shortcuts typed on Keyboard 2** (Settings → Raccourcis de profils): modifier + key, both
  on Keyboard 2 (e.g. Ctrl+F1 → Claude), from any window, also to leave Standard. Nothing is
  registered with Windows: Keyboard 1 keeps every combination. Defaults Ctrl+Alt+Shift+1..3 are
  proposed once for the first three macro profiles.
- **Profile line and page line** on the main screen; **drag and drop of a key onto another profile**
  (rest on the profile to show it); **multiple selection** (click, Shift+click, select all) and
  « Copier la sélection… » to copy several keys to another profile/page, all or nothing, with the full
  correspondence confirmed first.
- **Action sequences**: ordered steps (shortcut, key, write text, paste text, open program), each
  with its own delay; a *Record Shortcut* button per shortcut step. Limits: 50 steps, 60 s per
  delay, 120 s in total.
- **Printed sheet**: A4 landscape, action names under the keys, Arabic names fitted after the fonts
  load, Ctrl+P opens the preview; the action's type (Python/PowerShell) is never printed.
- **« Changer le clavier principal »** (Settings → Keyboard): every connected keyboard is shown with
  its role (clavier principal/contrôleur or clavier normal), **Utiliser comme clavier principal**,
  **Aucun contrôleur**, and **↶ Revenir au clavier précédent** (the previous controller is kept in
  the configuration as `previous_device`). Exactly one controller at a time; changing it never
  touches profiles, pages or keys; a keyboard without VID/PID is recognised by its exact path.
- **Development modes** without a build: `tools\run-dev.ps1` (`Test` = isolated copy, `--dry-run`;
  `Interactive` = isolated copy with real execution; `Real`), `--isolated` refuses the real folder,
  `tools\snapshot-dev.ps1` dated snapshots, `docs/DEVLOG.md`.
- Start-up check for the **WebView2 Runtime**: a clear message with the download link instead of a
  blank window (`--headless` skips it).
- `constraints.txt`: the exact package versions a release is built with (local build and CI).

### Changed
- **One copy at a time, whatever the copy**: besides the lock per installation folder, a lock per
  settings folder (an installed copy and a portable or source copy would overwrite each other's
  `config.json`) and one lock per session for the keyboard hook (two hooks ran every Keyboard 2
  action twice). `--dry-run` takes no hook lock. The second copy says which one is running.
- **The Windows-key block is withdrawn.** The unreleased development version blocked the Win key with
  a low-level hook (`WH_KEYBOARD_LL`), which cannot tell keyboards apart and therefore acted on
  Keyboard 1 too, against the rule that Keyboard 1 is never touched. The feature, its setting and its
  Settings switch are removed; Mythos Controller installs no low-level keyboard hook. A
  `block_windows_key` value in an existing `config.json` is ignored and dropped on load. Win
  shortcuts are still refused as profile shortcuts (Windows reserves the Windows key).
- `--dry-run` observes only: keys are delivered and nothing runs, not even a profile switch (the
  released 1.1.0 ran the action AND delivered the key: "0" + "ok" = "0ok").
- Saves caused by a profile/page switch rotate no backup (20 switches pushed every real edit out of
  the backup history).
- "Start with Windows" is reported enabled only when the entry points to this very copy; a stale
  entry from a moved or removed copy shows as disabled and is rewritten when re-enabled.
- The tray tooltip names the Standard profile instead of ON/OFF.

### Fixed
- A Win key *up* whose *down* was not swallowed is passed, so Windows can never be left with the
  Win key stuck down (every later key acting as Win+key) when the block is on.
- A Keyboard 2 modifier whose release was lost (keyboard unplugged, release in an elevated window)
  no longer turns a bare key into the profile shortcut forever: it is forgotten after 2 s.
- A profile created by the user under the name « Standard » in 1.1.x keeps its keys (renamed id
  `standard_user`); the application's Standard profile is added separately.
- Deleting a profile no longer edits the shortcut table in place while the input thread may read it;
  an unexpected error while deciding a Keyboard 2 key keeps its down and up consistent (swallowed).
- "Run command" shown in a console keeps a quoted path with spaces (`"C:\Program Files\..."`).
- The installed-application picker reads shortcut paths in UTF-8: shortcuts under a folder or a user
  name with an accent were silently dropped.
- Hook window accepts its message from normal applications when Mythos Controller runs elevated
  (UIPI); the installer starts the program with the user's own rights (`runasoriginaluser`).

### Compatibility
- `config.json` version stays 4: the new keys (`profile_hotkeys`, the `standard` profile, sequence
  step `program` and `delay_after_ms`) are optional. 1.1.x opens a 1.2.0 file (Standard active is
  written as OFF) but drops `profile_hotkeys` on its next save and refuses a sequence that contains a `program` step (it then starts from defaults
  and keeps the file as `config.invalid-*.json`). To go back to 1.1.x, restore `config.pre-1.2.0.json`
  or a backup first (`docs/RELEASE.md`).
- Tested on Windows 10 Pro 22H2 (build 19045) with the source version; the 1.2.0 installer and ZIP
  are built and smoke-tested by the release workflow (`docs/RELEASE.md`).

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
