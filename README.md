# مسار — Android (Capacitor) + Windows (Electron)

This project now builds for two platforms from the same app:

- **Android** — via Capacitor (unchanged from before).
- **Windows** — via Electron (new).

Both platforms load the exact same file, **`www/index.html`**, untouched.
Nothing about the app itself — tasks, habits, study timer, sleep tracking,
statistics, settings, the Arabic RTL layout, or the calm visual theme — was
rewritten or forked. Each platform is just a different native window/shell
around that one file.

```
masar-android/
├── www/
│   └── index.html            ← the app itself — shared by BOTH platforms, unchanged
├── icons/
│   ├── icon-source-1024.png  ← shared icon artwork (used to build both icon sets)
│   └── icon-512.png
├── package.json              ← Android/Capacitor project (unchanged)
├── capacitor.config.json     ← Android/Capacitor config (unchanged)
├── android/                  ← generated locally by `npx cap add android` (not included here)
├── electron/                 ← NEW — self-contained Windows desktop wrapper
│   ├── package.json          ← its own deps (electron, electron-builder) — separate from the root
│   ├── main.js                ← creates the desktop window, loads ../www/index.html
│   ├── .gitignore
│   └── build/
│       └── icon.ico           ← Windows app icon (same artwork as the Android icon)
└── README.md                  ← this file
```

**Isolation guarantee:** `electron/` has its own `package.json` and its own
`node_modules` when you `npm install` inside it. It never touches the root
`package.json`, `capacitor.config.json`, or the `android/` folder. Building
the Windows app cannot break or change the Android app, and vice versa.

---

## 1. Android — unchanged, still builds the same way

Nothing here changed from before. For completeness, the full steps again:

**Requirements:** Node.js (v18+), Android Studio (includes the Android SDK).

```bash
# from the masar-android/ root
npm install
npx cap add android      # generates android/ (first time only)
npx cap sync android
```

Add the app icon once (Android Studio → right-click `app/src/main/res` →
**New → Image Asset** → Launcher Icons (Adaptive and Legacy) → Foreground:
`icons/icon-source-1024.png` at ~70% scale → Background color `#6F8A6F`).

**Build the APK:**
```bash
cd android
./gradlew assembleDebug
```
or in Android Studio: **Build → Build Bundle(s)/APK(s) → Build APK(s)**.

**Where the APK appears:**
```
masar-android/android/app/build/outputs/apk/debug/app-debug.apk
```

**Install on your phone:** copy the APK to the phone (USB, Drive, etc.),
open it, allow installing from this source, tap Install. "مسار" appears in
the app drawer with its own icon and opens directly — no browser.

---

## 2. Windows — new, via Electron

Electron wraps `www/index.html` in a real native window (Chromium engine,
no browser UI, its own taskbar entry and `.exe`). The app's `localStorage`
persists inside that window's own local profile — closing and reopening
`Masar.exe` keeps all your data, exactly like the Android version keeps its
own data.

**Requirements:** Node.js (v18+). No Android Studio needed for this part.
electron-builder downloads Electron itself during `npm install`, so you'll
need an internet connection the first time.

### Build steps, from zero, to get `Masar.exe`

```bash
# 1. Go into the electron/ folder (NOT the project root)
cd masar-android/electron

# 2. Install Electron + electron-builder (isolated to this folder)
npm install

# 3. (Optional) Try it instantly without building an installer:
npm start

# 4. Build the actual Windows app
npm run dist:win
```

`npm run dist:win` must be run **on a Windows machine** (or with Wine
installed if cross-building from Linux/macOS) so the `.exe` is properly
packaged for Windows.

### Where the files appear

```
masar-android/electron/dist/Masar.exe              ← portable, standalone, no install needed
masar-android/electron/dist/Masar-Setup.exe        ← installer (adds Start Menu + Desktop shortcut, uninstaller)
masar-android/electron/dist/win-unpacked/Masar.exe ← unpacked build, useful for quick testing
```

- **`Masar.exe`** (portable) — copy this single file anywhere and double-click
  it. Nothing to install. This is the simplest option if you just want to run
  it on your own PC.
- **`Masar-Setup.exe`** — a proper installer: creates a Start Menu entry, a
  desktop shortcut, and an uninstaller, like a normal Windows program.

### What the window looks like

- Opens as its own window with the title "مسار" and the same calm sage/cream
  icon as the Android app — not inside any browser.
- Default size 520x860, resizable, with a minimum size (380x600) so it can
  never shrink below a usable layout.
- No browser-style menu bar (File/Edit/View) — it looks like a standalone
  program.
- The app's own CSS already has a `@media (min-width: 500px)` rule that
  centers the mobile-width card on a soft background once the window is
  wider than a phone — so on a desktop-sized window it automatically looks
  like a proper desktop app, framed nicely, with **no CSS changes needed**.

### Data persistence on Windows

`localStorage` inside an Electron window persists in that window's own
profile folder (under the user's `AppData`), completely separate from any
browser's storage. Closing and reopening `Masar.exe` keeps every task,
habit, study session, sleep entry, and setting.

---

## Files added or changed for this update

**Added (all new, nothing here existed before):**
- `electron/main.js`
- `electron/package.json`
- `electron/.gitignore`
- `electron/build/icon.ico`

**Unchanged (verified byte-identical, not touched):**
- `www/index.html`
- `package.json` (root)
- `capacitor.config.json`
- `icons/icon-source-1024.png`, `icons/icon-512.png`
- `.gitignore` (root)

**Updated (documentation only):**
- `README.md` (this file) — extended to cover both platforms.

---

## Limitations

- **This sandbox could not run either real build** — it has no internet
  access, no Android SDK, and no Windows machine to run `electron-builder`
  on. Both `.apk` and `.exe` must be produced by you, following the exact
  commands above, on your own machine.
- The Arabic font (**Tajawal**, via Google Fonts) is still loaded from the
  internet on first run, on both platforms. Offline on first launch, the app
  falls back to the system's default sans-serif font and still works fully.
  Bundling the font locally for a fully offline first run is a possible small
  follow-up, not done here since it requires a network fetch.
- Windows builds are **unsigned** (no code-signing certificate). Windows
  SmartScreen may show an "unknown publisher" warning the first time you run
  `Masar-Setup.exe` — click "More info → Run anyway". This is normal for an
  app without a paid code-signing certificate and doesn't affect
  functionality.
- No native plugins were added on either platform (no background
  notifications, no auto-update). The app's existing in-session behavior is
  unchanged.
