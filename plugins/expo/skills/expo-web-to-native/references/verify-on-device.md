# Verify a migrated screen against the running web app

Disclosed reference for [`expo-web-to-native`](../SKILL.md), steps 3–4. Verification means **running both apps and comparing the same screen** - the native port beside the web original. A clean compile or a green `expo export` proves nothing - a screen can build and still render blank or mis-render. This parity check is the gate the strangle loop runs each iteration.

## Choose available tools

Use existing browser and device capabilities that can open matching routes,
inspect rendered content, exercise interactions, and capture evidence. The recipe
below uses `agent-browser` and `argent`, but equivalent browser automation,
`agent-device`, `simctl`, `adb`, or manual on-device checks are valid. A separate
agent for each surface is optional.

- **Web example — `agent-browser`:** `open <url>`, `snapshot --json`, `screenshot <path>`.
- **Native example — `argent`:** `describe`, `debugger-component-tree`, gestures, and replayable `flow` checks. Invoke as `argent run <tool> --udid <udid>`; use `argent run list-devices` to find a device.

Check the chosen tools' availability and help before using their commands. Install
missing tools only when needed under the environment's installation permissions;
do not require a particular CLI when an available tool can perform the check.
If no device is available, continue independent implementation and report runtime
verification as blocked. Manual evidence must come from an actual device run;
source inspection cannot stand in for runtime behavior.

## The workflow

**A. Capture the web original** with agent-browser — the source of truth for parity. Run the web app (its `pnpm dev` server, or the **deployed URL** if local setup needs DB/auth env), then:

```bash
agent-browser open "<web-url>/<route>?<params>"
agent-browser snapshot --json      # structure to diff
agent-browser screenshot web.png   # visual reference
```

**Tip:** reuse captured baselines for the selected routes while the source and test data remain unchanged.

**B. Capture the native screen** (iOS shown via `simctl`; Android note below):
1. Run the app: `npx expo start --ios` (Expo Go). On **SDK 56+ both `@expo/ui` and DOM components run in Expo Go** — no dev build, no `react-native-webview` to install; reach for a dev build (the `expo-dev-client` skill) only for *custom* native modules. Stale-bundle trap: a CI-mode Metro + cached Expo Go can show an old build — terminate Expo Go and add `--clear` if a change doesn't appear.
2. Boot a sim: `xcrun simctl boot <udid>` (`xcrun simctl list devices available`); `open -a Simulator`.
3. Open the route: deep-link `xcrun simctl openurl booted "exp://<lan-ip>:8081/--/<route>?<params>"`, or argent `launch-app` + `gesture-tap`.
4. Capture: `xcrun simctl io booted screenshot native.png`, or `argent run describe --udid <udid>` for structure.

> **Android:** `simctl` / `expo run:ios` are iOS-only. Use an Android emulator + `adb` — `adb exec-out screencap -p > native.png`, `adb shell am start -a android.intent.action.VIEW -d "<deep-link>"`, `adb shell screenrecord` for motion — or `npx expo run:android` for a dev build.

**C. Compare** the two for the same route — layout, content, behavior. Compare rendered content and exercise matching interactions; use accessibility trees and screenshots as supporting evidence, allowing platform-specific structure. Pass only on parity: same data, and params passed into a DOM webview must produce the same result.

**Feel needs motion, not a still.** For a nativized screen with transitions, gestures, or haptics, a screenshot can't catch a janky push or wrong easing — capture a short recording (iOS `xcrun simctl io booted recordVideo feel.mov`; Android `adb shell screenrecord`; or an argent flow) and confirm it moves like a native app (see `native-patterns.md` → Feel).

## What "pass" looks like
- **DOM-shelled screen (step 3):** the web UI renders inside the native header/shell; params from the native route drive it the same as on web.
- **Nativized screen (step 4):** selected behavior and content match the source using native controls and interactions. Any retained DOM subtree is intentional and recorded as hybrid.
- **Blocked check:** record the route, missing access/tool/data, and remaining verification; do not mark it passed.
