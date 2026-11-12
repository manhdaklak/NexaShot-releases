# NexaShot - publish & install

Everything here is built from `..\src\ScreenUI`. Generated folders (`publish\`, `dist\`) can be deleted at any time.

| File | Purpose |
|---|---|
| `publish.ps1` | Builds a **self-contained** app (no .NET install needed on the target PC) into `publish\ScreenUI\` and a portable zip in `dist\` |
| `install.ps1` | Fallback per-user install: `%LOCALAPPDATA%\Programs\ScreenUI`, Start menu shortcut, entry in *Settings > Apps*. No admin rights. Does **not** allow-list the app in Controlled folder access by default (see below) |
| `uninstall.ps1` | Removes that per-user install (app, shortcuts, startup entry). Keeps captures and settings unless `-RemoveUserData` |
| `ScreenUI.iss` + `build-installer.ps1` | Recommended: builds a single `Setup.exe` with [Inno Setup 6 or 7](https://jrsoftware.org/isdl.php); **needs administrator rights, installs into `C:\Program Files\ScreenUI`**; optional code signing via env vars |

## Quick start

```powershell
cd installer
.\publish.ps1          # 1. build
.\install.ps1          # 2. install on this PC (and start the app)
```

Options: `.\install.ps1 -DesktopShortcut -StartWithWindows -NoLaunch`

If PowerShell blocks the scripts (execution policy):

```powershell
powershell -ExecutionPolicy Bypass -File .\publish.ps1
powershell -ExecutionPolicy Bypass -File .\install.ps1
```

## Giving it to other people

* **Portable zip** - `dist\ScreenUI-<version>-win-x64.zip`: unzip anywhere and run `ScreenUI.exe`.
  To install from the unzipped folder, copy `install.ps1` / `uninstall.ps1` into it and run `install.ps1`.
* **Setup.exe** - `winget install JRSoftware.InnoSetup`, then `.\build-installer.ps1`
  produces `dist\NexaShot-<version>-Setup.exe` (all users, `C:\Program Files\ScreenUI`, asks for administrator rights, optional
  desktop shortcut). "Start with Windows" is not an installer option: turn it on in the app's Settings (the app writes its own
  per-user Run key).

## Requirements on the target PC

Windows 10 version 2004 (build 19041) or newer, 64-bit. Nothing else. The app itself runs as a normal user (no elevation);
only `Setup.exe` (and its uninstaller) needs administrator rights.

## Tranh bi Windows chan (SmartScreen / Defender)

**Chi chung chi ky ma (code signing) moi het han canh bao.** Cac lua chon:

* Chung chi OV/EV tu CA (DigiCert, Sectigo, SSL.com...). EV co uy tin SmartScreen ngay; OV can tich luy danh tieng.
* **Azure Trusted Signing** (tra theo thang, re hon): can kiem tra dieu kien hien hanh (to chuc hay ca nhan duoc
  chap nhan, quoc gia, thoi gian xac minh). Neu dung: tao tai khoan + certificate profile tren Azure, cai
  `signtool` kem plugin Trusted Signing dlib, roi tu ky thu cong bang
  `signtool sign /fd SHA256 /tr http://timestamp.acs.microsoft.com /td SHA256 /dlib <Azure.CodeSigning.Dlib.dll> /dlib-metadata <metadata.json> <file>`
  (script `build-installer.ps1` chua ho tro truc tiep, chi co tai lieu).

Sau khi co chung chi, dat bien moi truong roi chay `build-installer.ps1` (khong co bien thi bo qua, chi canh bao vang):

```powershell
# Cach 1 (uu tien): chung chi trong kho CurrentUser\My
$env:NEXASHOT_SIGN_CERT_THUMBPRINT = '<thumbprint>'
# Cach 2: file pfx (import tam vao CurrentUser\My, ky bang thumbprint, roi xoa lai)
$env:NEXASHOT_SIGN_PFX = 'C:\path\cert.pfx'; $env:NEXASHOT_SIGN_PFX_PASSWORD = '<password>'
# Tuy chon
$env:NEXASHOT_SIGNTOOL = 'C:\...\signtool.exe'          # mac dinh: Windows SDK ban moi nhat
$env:NEXASHOT_TIMESTAMP_URL = 'http://timestamp.digicert.com'
.\build-installer.ps1
```

Script ky `ScreenUI.exe` truoc khi dong goi; `Setup.exe` va **uninstaller** duoc Inno Setup ky qua directive `SignTool`
(script truyen `/DSignEnabled=1 /Snexashot=...`, dung `/sha1 <thumbprint>`); zip portable duoc tao lai sau khi ky nen
exe trong zip da ky (khi khong ky thi `publish.ps1` tu zip nhu cu). Khong luu mat khau vao repo.
**Uu tien thumbprint**: voi PFX, mat khau doc tu bien moi truong (chi o dang SecureString trong script, khong dua len
command line signtool), cert duoc `Import-PfxCertificate` tam vao `CurrentUser\My` va **xoa** trong `finally`
(khong xoa neu cert da co san trong kho). Mat khau van nam trong bien moi truong cua phien, hay dat roi xoa sau khi build.
`publish.ps1` co them `-NoZip` / `-ZipOnly` (do build-installer dung). Version co hau to (`1.0.2-beta`) duoc rut thanh
`1.0.2` cho `VersionInfoVersion` (`/DAppVersionNumeric`).

* Neu Defender bao nham: gui file de Microsoft phan tich tai https://www.microsoft.com/wdsi/filesubmission
* Controlled Folder Access (CFA): chi cho phep theo **ung dung**, khong danh dau duoc thu muc, va can admin. **Setup.exe cai bang
  quyen admin vao `C:\Program Files\ScreenUI`** (`PrivilegesRequired=admin`, shortcut cho moi nguoi dung). Vi thu muc nay chi admin
  ghi duoc nen allow-list `ScreenUI.exe` la an toan: ma doc chay duoi quyen nguoi dung khong thay duoc exe/DLL (goi
  self-contained co `hostfxr.dll`, `coreclr.dll`...) de vuot CFA. Task "Allow NexaShot to save into protected folders" (mac dinh
  chon) chay truc tiep (Setup da elevated, khong them UAC) `{sys}\WindowsPowerShell\v1.0\powershell.exe -EncodedCommand ...`
  voi `Add-MpPreference -ControlledFolderAccessAllowedApplications '<duong dan ScreenUI.exe>'`. Khong co Defender hoat dong
  (AV ben thu ba: `Get-MpPreference` loi) -> thong bao "Microsoft Defender is not active; nothing to do"; CFA do GPO/Intune
  quan ly (co the bo qua allow-list cuc bo) hoac loi khac -> cai dat van thanh cong, chi hien thong bao ngan; app van tu
  chuyen sang `%LOCALAPPDATA%\ScreenUI\Captures` khi `Pictures` bi chan. Che do silent: van lam theo task nhung khong hien
  hop thoai. Duong dan duoc luu o `HKLM\Software\ScreenUI\CfaAllowedPath`; nang cap cung thu muc (cung duong dan) thi bo qua,
  khong chay lai. Go cai dat goi `Remove-MpPreference` (uninstaller da elevated nen silent cung chay) va **chi xoa co khi go
  thanh cong**. Khong them exclusion thu muc va khong tat CFA. App sau khi cai chay bang quyen nguoi dung thuong
  (Setup chay [Run] qua `explorer.exe`, xem muc Release process); Setup khong tao thu muc `Pictures\ScreenUI` (app tu tao/tu xu ly CFA) va khong ghi Run key
  (bat "Start with Windows" trong Settings cua app; HKCU cua admin elevate khac se sai).
* **Nang cap tu ban per-user cu** (Setup per-user truoc do hoac `install.ps1`): Setup tim ban cai trong HKCU cua nguoi dung
  dang chay Setup va hoi "An older per-user installation was found. Its shortcuts and uninstall entry will be removed ..."
  (silent thi tu lam khong hoi). Setup (dang elevated) **khong bao gio chay uninstaller cu hay uninstall.ps1 cu** (duong dan
  doc tu HKCU, ma doc cung user ghi duoc). No chi: xoa cac khoa Uninstall HKCU (`{AppId}_is1`, `ScreenUI`), xoa shortcut
  `NexaShot.lnk`/`ScreenUI.lnk` o Start menu/Desktop cua user (duong dan co dinh), dong process `ScreenUI.exe` neu image la
  `%LOCALAPPDATA%\Programs\ScreenUI\ScreenUI.exe`, va chi go entry CFA cu khi co HKCU `CfaAllowedPath` **bang dung** duong dan
  mac dinh do (khac thi bo qua va xoa co). Thu muc cai cu khong bi xoa o buoc nay: NexaShot tu xoa o lan chay dau (quyen
  thuong). Captures va settings
  (`Pictures\ScreenUI`, `%APPDATA%\ScreenUI`) khong bi xoa. **Gioi han**: neu Setup duoc elevate bang tai khoan admin khac thi
  HKCU la cua admin do va khong thay ban cu cua nguoi dung kia (go thu cong trong Settings > Apps). Khoa Run cu bi go cung ban
  cu nen can bat lai "Start with Windows" trong Settings.
* **`install.ps1` (per-user)**: `-AllowControlledFolderAccess` nay mac dinh **tat**. Bat no tren ban cai trong
  `%LOCALAPPDATA%` lam yeu CFA (ma doc thay duoc exe da duoc allow-list); hay dung Setup.exe. Neu bat, co `Remove-MpPreference`
  trong `uninstall.ps1` (flag o `HKCU\Software\ScreenUI`; chi go khi flag bang dung duong dan exe cua ban dang go, chi xoa flag khi go thanh cong). Cac script goi PowerShell bang
  duong dan tuyet doi + `-EncodedCommand`.

## Changing the version

Edit `<Version>` in `src\ScreenUI\ScreenUI.csproj`; the scripts and installer pick it up.

## Release process (in-app update)

1. Raise `<Version>` in `ScreenUI.csproj`.
2. `.\build-installer.ps1` creates `dist\NexaShot-<ver>-Setup.exe` and `dist\NexaShot-<ver>-Setup.exe.sha256`
   (64 lowercase hex characters, no file name, no newline; computed after signing).
3. Create a GitHub release `v<ver>` on `manhdaklak/NexaShot-releases` and attach **both** files.
   The public release repo's `README.md` is the template `release-repo\README.md` (paste it once, update when terms
   change); `release-repo\EULA.txt` is the same Terms of use, shown as the license page in Setup (`LicenseFile`;
   silent updates are not blocked). Both are template text, not legal advice.

Third-party licenses: `THIRD-PARTY-NOTICES.txt` (+ `notices\dotnet-ThirdPartyNotices.txt`) are copied by `publish.ps1` into
the app folder, so the installer (also a Start menu entry) and the portable zip include them. The texts were copied verbatim
from the .NET 10.0.14 runtime packs and the NAudio 2.2.1 package in `%USERPROFILE%\.nuget\packages`; when the runtime or
NuGet packages change, re-check them (the Windows SDK projection package has only a license URL).

The app downloads both, checks the SHA-256, then runs Setup with `/SILENT /SUPPRESSMSGBOXES /NORESTART /CLOSEAPPLICATIONS`
(UAC prompt, because Setup needs administrator rights). Setup closes the running `ScreenUI.exe` through Restart Manager
(default `CloseApplicationsFilter` covers it; if it cannot be closed, Inno's default applies: files replaced at next reboot).
`RestartApplications=no` (Restart Manager could relaunch the app elevated); instead a `[Run]` entry with `Check: WizardSilent`
starts the app again through `explorer.exe "<app>\ScreenUI.exe"` (the postinstall entry does the same). Reason: the app starts
Setup with `Verb runas`, so Setup's "original user" token is already elevated and `runasoriginaluser` alone cannot drop the
rights; Explorer starts the app with the normal shell token. In silent mode the `cfa` task only runs when the
`HKLM` flag differs from the exe path (so normally not on updates) and shows no message boxes.

## Good to know

* The app is **not code-signed** unless you configure a certificate (see below), so Windows SmartScreen may show
  "Windows protected your PC" the first time (More info > Run anyway).
* User data lives outside the install folder, so upgrades and uninstall keep it:
  captures in `Pictures\ScreenUI` (changeable in Settings), settings in `%APPDATA%\ScreenUI\settings.json`,
  custom stamps in `%APPDATA%\ScreenUI\Stamps`.
* Company security tools (Defender for Endpoint) may flag screen-capture APIs on first run of an unsigned build.
