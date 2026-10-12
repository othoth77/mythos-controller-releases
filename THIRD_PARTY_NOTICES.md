# Third-party notices

Mythos Controller is released under the MIT License (see `LICENSE`).

This file has two parts: (1) projects whose techniques or code Mythos Controller
adapts, and (2) the third-party components that are **bundled in the released
executable** (`MythosController.exe` and its `_internal` folder). The full text of every
bundled component's license is in the `licenses/` folder, shipped next to the executable.

# Part 1 - Adapted projects

Mythos Controller builds on techniques and, in places, adapted code from the following
MIT-licensed open-source projects. Their copyright notices are reproduced
here as required by the MIT License.

## BindBoard (foundation)

- Repository: https://github.com/Mark-mark-228/BindBoard
- License: MIT (declared in the project README; the repository ships no
  separate LICENSE file, which is recorded in `docs/FOUNDATION_DECISION.md`)
- What was reused: the overall driver-free architecture (Windows Raw Input
  for device identification combined with a `WH_KEYBOARD_LL` hook for
  suppression), the ctypes declarations for the Raw Input structures, the
  Raw Input device enumeration approach, the pywebview + pystray application
  shell pattern, and the foreground-process lookup.
- Adapted into: `src/mkc/input/win32_types.py`, `src/mkc/input/rawinput.py`,
  `src/mkc/input/hook.py`, `src/mkc/context/foreground.py`, `src/mkc/tray/`.

```
MIT License
Copyright (c) 2026 Mark-mark-228 (BindBoard)
```

## DeviceMapper

- Repository: https://github.com/HiMindAi/DeviceMapper
- License: MIT, Copyright (c) 2026 HiMindAi
- What was reused: the hot-plug pattern (`RIDEV_DEVNOTIFY` +
  `WM_INPUT_DEVICE_CHANGE` with device-cache refresh), the VID/PID parsing of
  Raw Input device paths, and the foreground-context matching concept
  (`exe`, `!exe`, `any`).
- Adapted into: `src/mkc/input/rawinput.py`, `src/mkc/devices/identity.py`,
  `src/mkc/context/rules.py`.

```
MIT License
Copyright (c) 2026 HiMindAi
```

## HIDeous

- Repository: https://github.com/dero/HIDeous
- License: MIT, Copyright (c) 2025 Jaroslav Polakovič
- What was reused: the device-attribution technique (a global `WH_KEYBOARD`
  hook DLL that asks the main application, synchronously, whether to block a
  key, after Raw Input has named the device; a shared "interesting keys"
  filter so other keys never leave the target application), the `SendInput` +
  `KEYEVENTF_UNICODE` text injection approach including UTF-16 surrogate-pair
  handling and the injected-event marker, and binding by physical scan code
  instead of virtual key. Re-implemented: the hook DLL in C
  (`src/mkc/native/mkc_hook.c`), the rest in Python.
- Adapted into: `src/mkc/native/`, `src/mkc/input/keyboard_backend.py`,
  `src/mkc/actions/text_injection.py`, `src/mkc/input/scancodes.py`.

```
MIT License
Copyright (c) 2025 Jaroslav Polakovič
```

## Runtime dependencies

| Package | License | Purpose |
|---|---|---|
| pywebview | BSD-3-Clause | Application window (WebView2) |
| pystray | LGPL-3.0 (dynamically imported, unmodified) | System tray icon |
| Pillow | MIT-CMU (HPND) | Tray icon rendering |
| pythonnet / clr_loader | MIT | Required by pywebview on Windows |
| PyInstaller | GPL-2.0 with bootloader exception (build-time only) | Windows build |
| ziglang (Zig toolchain) | MIT (build-time only) | Compiles the hook DLLs |
| pytest | MIT (dev only) | Tests |

# Part 2 - Components bundled in the released executable

| Component | Version | License | Used for | License text |
|---|---|---|---|---|
| Python runtime | 3.14 | PSF-2.0 | interpreter embedded by PyInstaller | `licenses/Python-PSF-LICENSE.txt` |
| pywebview | 6.2.1 | BSD-3-Clause | the application window (WebView2) | `licenses/pywebview-LICENSE.txt` |
| pystray | 0.19.5 | **LGPL-3.0** | system tray icon | `licenses/pystray-COPYING.LGPL.txt`, `licenses/pystray-COPYING.txt` |
| Pillow | 12.3.0 | MIT-CMU (HPND) | tray icon drawing | `licenses/Pillow-LICENSE.txt` |
| six | 1.17.0 | MIT | dependency of pystray | `licenses/six-LICENSE.txt` |
| bottle | 0.13.4 | MIT | pywebview's local page server | `licenses/bottle-LICENSE.txt` |
| proxy_tools | 0.1.0 | MIT (declared in the package metadata; the package ships no license file) | dependency of pywebview | - |
| pythonnet | 3.2.1 | MIT | .NET bridge used by pywebview | `licenses/pythonnet-LICENSE.txt` |
| clr_loader | 0.3.1 | MIT | dependency of pythonnet | `licenses/clr_loader-LICENSE.txt` |
| cffi | 2.1.1 | MIT-0 | dependency of clr_loader | `licenses/cffi-LICENSE.txt` |
| pycparser | 3.11 | BSD-3-Clause | dependency of cffi | `licenses/pycparser-LICENSE.txt` |
| typing_extensions | 4.16.0 | PSF-2.0 | dependency of pywebview | `licenses/typing_extensions-LICENSE.txt` |
| Microsoft.Web.WebView2 (SDK DLLs shipped by pywebview) | 1.0.3856.49 | BSD-style, Copyright (C) Microsoft Corporation | WebView2 .NET wrapper and loader | `licenses/Microsoft-WebView2-SDK-LICENSE.txt` |
| IBM Plex Sans, Sans Arabic, Mono (web fonts) | - | SIL OFL 1.1 | interface typography | `licenses/IBM-Plex-OFL-1.1.txt` |
| Archivo Expanded (web font) | - | SIL OFL 1.1 (Copyright 2019 The Archivo Project Authors, https://github.com/Omnibus-Type/Archivo) | interface titles | same OFL text |

The Microsoft Edge **WebView2 Runtime** itself is not bundled: it is a Windows component
(installed with Windows 11 and with Microsoft Edge on Windows 10). Mythos Controller uses it
as installed on the machine. The Microsoft Visual C++ runtime DLLs next to the executable are
redistributed under Microsoft's redistribution terms.

## pystray and the LGPL-3.0

pystray is used **unmodified**, as an ordinary Python package in the `_internal` folder of the
one-folder build (not merged into the executable file). You may replace it with another
version of pystray (for example one you modified) by replacing the `pystray` package files in
`_internal`. The source of pystray is at https://github.com/moses-palmer/pystray. Mythos
Controller's own code is MIT and is not a derivative of pystray.

## Fonts

The font files are the unmodified web fonts (`.woff2`) of the projects above. The OFL's
"Reserved Font Name" conditions are respected: the fonts are not renamed, modified or sold
separately.

