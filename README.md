# Mythos Controller - downloads

Turn a **second physical keyboard** into a dedicated macro keyboard on Windows 10 / 11, without
drivers. Your normal keyboard (Keyboard 1) is never touched; every key of the second keyboard
(Keyboard 2) can type text, paste a snippet, send a shortcut, open a program or a website, run a
command or your own Python / PowerShell code, or switch profiles and pages.

This repository only distributes the **released builds**, the install instructions and the changelog.
The source code is developed in a private repository.

## Download

**[Latest release](https://github.com/othoth77/mythos-controller-releases/releases/latest)** - every release has:

| File | What it is |
|---|---|
| `MythosController-<version>-win64-setup.exe` | **Installer** (recommended). Per-user, no administrator rights needed. |
| `MythosController-<version>-win64.zip` | Portable build: unzip and run `MythosController.exe`. |
| `SHA256SUMS.txt` | Checksums of the two files above. |

No GitHub account or permission is needed to download.

## Install

1. Download the installer from the latest release.
2. *(Recommended)* Check it: in PowerShell, `Get-FileHash .\MythosController-<version>-win64-setup.exe` must print the same
   SHA-256 as the matching line of `SHA256SUMS.txt`.
3. Run the installer. Windows SmartScreen may say the publisher is unknown, because the files are **not
   code-signed**: choose **More info -> Run anyway** only if the checksum matched.
4. Start **Mythos Controller** from the Start menu. In **Devices**, click **Use as Keyboard 2** on your second keyboard,
   then click a key on the on-screen keyboard and choose what it does. Drag a key onto another key to copy its setting.

**Update**: run the newer installer; it installs over the old version. **Uninstall**: Windows Settings -> Apps.
Your settings, backups and logs live in `%APPDATA%\MythosKeyboardController` and are **never** touched by an
install, update or uninstall (delete that folder yourself if you want them gone).

## Requirements

- Windows 10 or 11, 64-bit.
- Microsoft Edge **WebView2 Runtime** (included with Windows 11 and with Microsoft Edge on Windows 10).
- Python (or the `py` launcher) **only** if you use Python *Custom Code* keys; PowerShell code works as is.

## Known limitations and security notes

- **Not code-signed**: SmartScreen may warn on first run; some antivirus tools are cautious about any program that installs a
  keyboard hook (this one is a documented `WH_KEYBOARD` hook used to tell the two keyboards apart).
- The hook does not reach **elevated (administrator) windows** or Microsoft Store (UWP) apps: there, a Keyboard 2 key
  types normally and its action does not run.
- With Keyboard 2 switched ON every key of Keyboard 2 is swallowed; keys without a setting do nothing.
- **Custom Code** runs with your user rights and is not sandboxed. It only runs after you have checked and *activated* it key by key.
- By default logs contain no typed text and no tokens, and links are reduced to `scheme://host` (typed text is logged only if you switch "log sensitive" on).
- Tested on Windows 10 Pro 22H2. Windows 11 is expected to work but was not tested.
- Going back to 1.0.0 is not supported if you use Custom Code keys (back up `%APPDATA%\MythosKeyboardController` first).

See [CHANGELOG.md](CHANGELOG.md) for what changed in each version and [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)
for the open-source components (the full license texts are installed in the `licenses` folder next to the program).

## Support

Open an [issue](https://github.com/othoth77/mythos-controller-releases/issues). Do not paste configuration files, logs or
credentials into a public issue.

## License

Mythos Controller is released under the [MIT License](LICENSE).
