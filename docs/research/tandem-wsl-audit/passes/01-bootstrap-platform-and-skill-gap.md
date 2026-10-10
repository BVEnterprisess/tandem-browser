# Pass 01/12: Bootstrap, platform, and canonical layout

Source: `main` commit `3b613cfd4c299609ca7ca415d638c1b71c6ba5de`. Source-only research; no runtime execution or tests.

## Sources and method

Read `AGENTS.md`, `CLAUDE.md`, `PROJECT.md`, `ARCHITECTURE.md`, `TODO.md`, `README.md`, `package.json`, `docs/platform-support.md`, `src/main.ts`, `src/registry.ts`, `src/platform/index.ts`, `src/platform/capabilities.ts`, `src/platform/stealth-ua/index.ts`, `src/platform/paths/index.ts`, `src/platform/tests/platform.test.ts`, `src/sessions/manager.ts`, `src/utils/constants.ts`, `scripts/start.js`, and `tests/smoke/startup-smoke.js`.

Read Matt Pocock skill source from `BVEnterprisess/skills`: setup, domain-modeling, codebase-design, writing-for-agents, wayfinder, research, plus setup templates. Read Ponytail, Ponytail-audit and Ponytail-review source from `runaki-ai`. The runtime skill registry listed no installed skills; retrieved sources are not proof of installed versions. Applied source-guided gap audit and complexity deletion test, not a runtime skill invocation.

## Verified findings and proposals

**01: Linux persona.** `src/platform/index.ts` selects the Darwin UA adapter on Linux; `src/platform/tests/platform.test.ts` explicitly expects `macOS` client hints for Linux. `src/platform/capabilities.ts` calls Linux support partial. Verified source/test expectation, not an observed browser run. Propose a future compatibility evaluation, not an immediate change.

**02: WSL2 support status.** `docs/platform-support.md` designates Linux x64 best-effort and Windows 11 x64 Tier 1. WSL2 follows the Linux adapter. Microsoft documents WSLg on Windows 10 build 19044+; build 19045 satisfies that prerequisite, not a guarantee of correct Electron/GPU operation. Primary source: https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps.

**03: Launcher.** `scripts/start.js` starts Electron without an explicit WSLg display preflight. It also forcibly terminates identified leftover Unix Electron processes before relaunch. `src/utils/constants.ts` sets a 30-second cookie flush interval. Session-loss risk is hypothetical; no loss was observed.

**04: Named sessions.** `src/sessions/manager.ts` initializes a process-local Map with default only. Named partitions use the `persist:` prefix, but the name registry is not serialized in that module. Investigate whether other modules restore registrations before concluding a restart defect.

**05: Memory and security.** `src/main.ts` sets a global 4096 MB V8 heap ceiling; this is not actual memory allocation. The same file enables sandbox and context isolation and disables renderer Node integration. Preserve authenticated state and sandbox. No WSLg/GTX1660 memory benchmark performed.

**06: Tests.** `tests/smoke/startup-smoke.js` polls the HTTP status endpoint, not visible GUI, renderer acceleration, authenticated restart, or Midscene. Tests were read, not run.

## Ponytail-only complexity observation

`shrink:` `scripts/start.js` invokes a cache-cleanup function with a hardcoded macOS home path on every OS. It normally returns without action on Linux. A platform guard is a candidate, pending macOS regression evidence. No dependency removal or measurable net line reduction established. Correctness and performance findings above are outside Ponytail's complexity-only remit.

## Matt Pocock read-only gap

`CLAUDE.md` exists and points to `AGENTS.md`; setup skill therefore targets CLAUDE.md, not AGENTS.md. Exact-path reads found no root GLOSSARY.md, CONTEXT.md or GLOSSARY-MAP.md, and no docs/agents/{issue-tracker,domain,triage-labels}.md. GitHub is the repo host. Issue labels and ADR inventory not established. Prefer single-context glossary created lazily for resolved vocabulary, not duplicate context scaffolding. Upstream repository references remain in AGENTS.md and package.json; separate attribution from fork distribution decisions. See `../proposals/matt-pocock-canonicalization.md`.

## Next

Trace session restoration, cookie flushing, tab bootstrap, and API routes. Midscene seam and live Chrome-profile reuse are unverified. No implementation changes authorized.
