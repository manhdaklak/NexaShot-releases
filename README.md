# NexaShot

NexaShot is a Windows screen capture, image annotation and screen recording tool that lives in the system tray.
This repository only hosts the downloadable releases.

## Download

Get the latest `NexaShot-<version>-Setup.exe` from the
[latest release](https://github.com/manhdaklak/NexaShot-releases/releases/latest).

- Requires Windows 10 (build 19041 or newer), 64-bit.
- Setup needs administrator rights (you will see a UAC prompt) and installs into `C:\Program Files\ScreenUI`.
  The app itself runs as a normal user.
- **SmartScreen warning:** the installer is not code-signed yet, so Windows may show "Windows protected your PC".
  Click **More info**, then **Run anyway**.
- **Verify the download (optional).** Each release has a `.sha256` file next to the Setup. In PowerShell:

  ```powershell
  (Get-FileHash .\NexaShot-<version>-Setup.exe -Algorithm SHA256).Hash.ToLower()
  Get-Content .\NexaShot-<version>-Setup.exe.sha256
  ```

  The two values must be identical.

## Updating

NexaShot checks for updates itself and can install them for you. You can also download a newer Setup and run it over
the existing installation. Your captures and settings are kept.

## Terms of use

NexaShot is freeware. Copyright (c) 2026 manhdaklak. All rights reserved.

- You may download and use it free of charge, for personal or work purposes.
- You may **not** redistribute, resell, modify or reverse engineer it, except where applicable law expressly allows it.
- It is provided "AS IS", without warranty of any kind. The author is not liable for any damages arising from its use.

The same terms are shown in the installer (`EULA.txt`).

## Privacy

NexaShot checks GitHub for updates (enabled by default; can be turned off in Settings).

NexaShot does not collect or send your data. Your captures, recordings and settings stay on your PC. The only network
connection is to GitHub, to check for updates (when update checking is enabled).

## Third-party software

NexaShot includes third-party components (.NET, NAudio and others) under their own licenses. See
`THIRD-PARTY-NOTICES.txt` in the installation folder.

<!-- Template text, not legal advice. Have it reviewed if you need legal certainty. -->
