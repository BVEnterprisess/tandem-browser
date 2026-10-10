# Pass 03/12: Electron lifecycle, launcher, and Linux/WSL startup

Source: `BVEnterprisess/tandem-browser` at `3b613cfd4c299609ca7ca415d638c1b71c6ba5de`. Read-only source inspection on 2026-10-10. Tests **NOT EXECUTED**. This is an independent lifecycle module, not a completion claim for missing pass 02.

## Concrete startup sequence

1. `package.json` scripts `start` and `dev` invoke `node scripts/start.js`. `scripts/start.js:315-325` runs `runCompile()`, `clearMacOSQuarantine()`, `cleanupApiPort()`, `clearServiceWorkerScriptCache()`, then `startElectron()`. `runCompile` invokes `npm run compile` unless `--skip-compile` is supplied (`scripts/start.js:135-145`). `package.json` requires Node >=20 and declares Electron ^41.10.3 and native `better-sqlite3` with an `electron-rebuild` postinstall.
2. `scripts/start.js:277-312` resolves `node_modules/electron/dist/electron` on Linux, spawns it with `['.']`, inherited stdio, repo CWD and cleaned Electron-as-Node environment flags. SIGINT/SIGTERM received by the launcher are forwarded to the Electron child; the launcher exits with the child exit code.
3. `src/main.ts:952-969` waits for Electron `app.whenReady()`, creates a window, starts the API and constructs the menu. `src/main.ts:818-855` selects platform window options and creates a sandboxed `BrowserWindow` with `contextIsolation: true`, `nodeIntegration: false`, and `partition: DEFAULT_PARTITION`; the shell is loaded from `shell/index.html`.
4. `src/main.ts:868-894` initializes runtime managers and starts the HTTP API. `src/api/server.ts:625-651` binds the configured `apiListenHost`, defaulting to `0.0.0.0`, and rejects on server errors, including `EADDRINUSE`. This is an observed source default, not a verified network exposure measurement.
5. `src/main.ts:939-947` registers initial tab lifecycle. `src/bootstrap/tab-session.ts:17-59` loads tab restore data and opens restored tabs in the default `persist:tandem` partition. Session/partition fidelity is already an unresolved separate pass-02 finding.

## Shutdown and restart behavior

- `src/main.ts:857-864` calls `teardown()` on window close; `src/main.ts:994-1002` calls it again on `will-quit` and quits when all windows close on non-macOS. `teardown()` clears the cookie interval and invokes `destroyRuntime` (`src/main.ts:219-233`). `destroyRuntime` calls `api.stop()` and destroys managers (`src/bootstrap/runtime.ts:521-562`); `api.stop()` calls asynchronous MCP transport stop without awaiting it and closes the HTTP server (`src/api/server.ts:656-661`). No explicit completion barrier for asynchronous shutdown was observed in these paths.
- `src/main.ts:552-555` periodically invokes `ses.cookies.flushStore()` for the **default** session. `clearCookieFlushTimer()` clears the interval on shutdown, but these inspected shutdown paths do not explicitly await a final `flushStore()`. This establishes a missing explicit barrier in the inspected path, **not observed cookie loss**.
- `src/tabs/manager.ts:605-625` defers tab-session serialization by 500 ms and states that timers are cleared naturally; the inspected `destroyRuntime` does not call a tab-manager flush method. A close within the debounce window may leave the latest tab list unsaved; this is a source-derived risk requiring runtime reproduction.
- `scripts/start.js:157-186,236-244` checks the API port on Linux/macOS using `lsof` and `ps`; if the process command contains Electron/Tandem and the repository root, it invokes `kill -9` on the matching PID before starting. Unknown owners cause an error rather than a kill. This is a confirmed force-kill branch. Its effect on durable browser state is **not tested**. `lsof` errors are not separately checked before an empty-PID return, so missing `lsof` does not establish an explicit preflight failure.
- `scripts/start.js:247-274` calls a cache cleanup helper on all platforms, but its target path is hardcoded under macOS `~/Library/Application Support/tandem-browser`. The path normally will not exist in WSL2; this is a narrow `shrink:` candidate, **not** justification to remove service-worker behavior or any dependency.

## Linux/WSL boundary and limits

- `src/platform/capabilities.ts` labels Linux x64 `tier2-best-effort`, with partial source startup; `src/platform/index.ts` selects Linux paths, process and window adapters. `src/platform/paths/index.ts:15-36` resolves Linux Tandem data to `~/.tandem`.
- No WSLg `DISPLAY`, `WAYLAND_DISPLAY`, `XDG_RUNTIME_DIR`, graphics-driver, GPU acceleration, or sandbox compatibility preflight exists in the inspected launcher sequence (`scripts/start.js:315-325`). Absence of preflight does **not** prove WSLg incompatibility.
- `src/main.ts:22-33` sets a 4096 MB V8 old-space ceiling and disables several Chromium features. This is not observed RAM allocation or GTX1660 GPU utilization.
- The target Windows 10/WSL2/GTX1660 environment was not accessed. No GPU, GUI, memory, display, restart, or cookie-durability test was executed.

## Existing tests and missing evidence

- `tests/smoke/startup-smoke.js:1-160,170-235` spawns the launcher with `--skip-compile`, polls the HTTP `/status` endpoint, then sends SIGTERM and escalates to SIGKILL after five seconds. It does not assert a visible WSLg window, hardware acceleration, a graceful persisted restart, or preservation of named authenticated partitions.
- `src/platform/tests/platform.test.ts:90-117` checks Linux adapter selection, data paths and window options, not actual WSLg startup. **All tests inspected, none executed.**

## Evidence-gated acceptance vectors for future implementation

1. With an occupied API port owned by an unrelated process, launcher aborts without killing it; with a stale Tandem process, record whether force-kill is necessary and whether tab/cookie state survives. Do not run against a real authenticated profile.
2. Close the app within 500 ms of a tab change; restart and compare saved tab state. Separately verify named-partition continuity.
3. After a cookie mutation, close before the 30-second timer; restart and verify persistence without exporting credentials into test logs.
4. On the target WSL2 system, check Electron GUI creation, API readiness, display backend, sandbox behavior, memory usage and GPU acceleration independently. HTTP `/status` alone is insufficient.
5. Validate that lifecycle cleanup completes without unhandled rejection, and that normal shutdown does not require SIGKILL.

## Ponytail application and boundaries

Read the published `ponytail`, `ponytail-audit`, and `ponytail-review` SKILL.md files from `marmelab/atomic-crm/.claude/skills/`. The installed skill registry returned no entries, so these are retrieved instructions, **not verified locally installed versions**. Applied complexity-only tags: `shrink:` a macOS-specific cache cleanup is invoked on Linux despite an OS-specific hardcoded path (`scripts/start.js:247-274`). No measured line reduction or removable dependency is established. Correctness, durability, and security observations above are deliberately separate from complexity review.

## Persistence and next

Pass 02 `passes/02-session-identity-persistence-restart.md` remains absent at the beginning of this investigation; do not mark it completed without verified commit/readback. Next independent module: Linux graphics/display/runtime prerequisites, including Electron sandbox and GPU compatibility. Research branch only; no production modifications.
