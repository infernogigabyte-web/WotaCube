#  WotaCube

**WotaCube** is an Electron-based speedcubing application. It features a dedicated timer, an algorithm trainer, and statistics — all in one place.

The core architectural principle of the project is a **complete rejection of third-party JavaScript libraries**. The entire frontend is written strictly in pure Vanilla JS, CSS, and HTML, which guarantees instant loading, minimal bundle size, and maximum interface responsiveness.

---

##  Features

###  Timer Control
* Hold the timer to activate it (or hold the **Spacebar** on your computer).
* Release to start the timer, and press again to stop.
* Easily swipe left or right to switch tabs.
* **Hybrid Timing Mechanism:** The live UI timer display utilizes `requestAnimationFrame` + `performance.now()` for fluid animations. For the desktop Electron build, the final result (`solve`) is captured using a **native C++ timer** (`std::chrono::steady_clock`) via IPC, eliminating any main-thread JavaScript lag. PWA build uses `performance.now()` as a fallback.

###  Quick Panel
Right after a solve, a quick panel pops up allowing you to:
* Delete the result.
* Mark it as `+2` or `DNF`.
* Close the panel.
* **Undo feature:** Accidentally deleted a result? You can undo the action with a single click.

###  Sessions & Stats
* Create separate sessions for different puzzles or training styles.
* Your `ao5`, `ao12`, and Personal Best (PB) are always visible.
* Complete statistics with progress charts (Catmull-Rom curve) and date range filtering.
* **WCA-Compliant:** Averages are calculated automatically using official WCA trimming rules.

###  Algorithm Trainer & Modes
* **Practice Mode:** Train specific cases or entire subsets. These attempts are excluded from session stats to keep your `ao5` and PB clean.
* **Competition Mode:** Simulates real-life competitions by pre-generating scrambles and locking the results screen until the last solve.
* **BLD Mode (Blindfolded):** Features a dedicated two-phase timer for memorization (`memo`) and execution (`solve`).
* **Stackmat Mode:** Interactive visual simulation of a physical Stackmat timer.
* **Built-in Database:** Contains **241 base cases** and community alternatives covering **F2L/Advanced F2L, OLL, PLL, VHLS, Winter Variation, COLL, and OLL-CP**.

### Customization & UI
Make the app yours in the Settings section:
* **Themes and Accent Colors:** Includes Dark, Light, and Auto (time-based) modes with 6 accent choices. Both primary themes use custom-tailored warm tones to drastically reduce eye strain during long sessions.
* **Custom UI Elements:** Native OS scrollbars are completely hidden and replaced with a custom JS implementation for platform uniformity. Features 3D-tilt card effects and smooth "Pixel transitions" between screens.
* **Vibration Feedback & Sound:** Toggleable audio profiles and haptics.
* **Display Settings:** Full-screen mode, focus mode, and timer precision toggle (2 or 3 decimals).
* **Support for 6 Languages:** English, Russian, Chinese, Hindi, Spanish, and Arabic (with localized layout handling to preserve UI structure).
* **Accessibility:** All custom modals and dialogs feature a robust keyboard focus trap (`getModalFocusable`).

---

## Tech Stack & Architecture

### Platforms
The application builds from a single codebase for three target environments:
1. **Web / PWA:** A static website capable of running fully offline via a configured Service Worker (`sw.js`).
2. **Desktop (Electron):** A cross-platform desktop application built using `electron-builder`.
3. **Native Modules:** Low-level extensions in C++ and Rust (`napi-rs`) for desktop-specific enhancements.

### File Structure
* `index.html`, `manifest.json`, `sw.js` — PWA shell wrapper and service worker.
* `css/` — Stylesheets and theme management engine.
* `js/` — Modular core (UI orchestration, timer logic, scramble generation, i18n, and storage handling).
* `data/` — Local database files for algorithms and translation files.
* `main.js`, `preload.js` — Entry points and secure communication bridges for Electron.
* `native/`, `native-rust/` — Native module source code.

###  Data Management & Resilience
* **Storage:** Session state is managed via `localStorage`.
* **Corruption Protection:** If a JSON parsing failure or outdated schema is detected, `storage.js` initiates automatic field migrations. In the event of a critical failure, legacy data is safely archived under a unique backup ID instead of being overwritten.
* **Import/Export:** Full support for application-wide backups and individual session importing/exporting using CSV files.

###  Security & Electron Process Isolation
* **Sandbox Security:** The renderer process is completely isolated and sandboxed:
  ```javascript
  contextIsolation: true, nodeIntegration: false, sandbox: true
  ```
* **IPC Bottlenecking:** `preload.js` exposes exactly 4 synchronous IPC calls (`timer` and `scramble` actions). Synchronous execution prevents race conditions when registering critical solve timestamps.
* **Single-Instance Lock:** Prevents simultaneous instances from silently overwriting shared `localStorage` data by shifting focus to the already running window. External link clicks are forced out of the sandbox into the default system browser.
* **Graceful Degradation:** If native binaries fail to compile for a specific platform architecture, the app seamlessly falls back to regular browser APIs without crashing.

---

*Good luck chasing your PB!* 🚀
