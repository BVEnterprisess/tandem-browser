# Pass 08/12: Extensions, native messaging, and WSL2 boundary

Repository: `BVEnterprisess/tandem-browser`. Pinned implementation commit: `3b613cfd4c299609ca7ca415d638c1b71c6ba5de`. Source-only investigation, 2026-10-11. Tests executed: **0**. No installation, runtime, or implementation change.

## Continuity and provenance

Read committed `INDEX.md`, Pass 01, Pass 03, and the canonicalization proposal; read prior locally saved Pass 05-07 investigations. Pass 02/04 remain uncommitted; Pass 05-07 remain local-only. This pass covers a new module, not those earlier topics.

Installed skill registry: **empty**. Retrieved and read published `ponytail`, `ponytail-audit`, and `ponytail-review` SKILL.md from `marmelab/atomic-crm/.claude/skills/`; these are not confirmed installed versions. Applied the complexity-only deletion/native/stdlib/YAGNI tests separately from correctness and security.

## Verified source findings

**F08-01: Native host discovery and actual proxy lookup disagree on Linux.** `src/platform/native-messaging/index.ts:149-156` supplies Linux manifest directories under `/etc/opt/chrome/native-messaging-hosts`, `~/.config/google-chrome/NativeMessagingHosts`, and `~/.config/chromium/NativeMessagingHosts`. `src/extensions/manager.ts:135-143,175-177` uses the selected platform adapter to detect hosts. But the actual HTTP/WebSocket proxy's `findHostManifest()` in `src/extensions/nm-proxy.ts:68-107` searches only hard-coded macOS directories, ignoring that adapter. Its routes call this finder at lines 304 and 381. Therefore a Linux-only manifest can be detected by the status layer but not resolved by the proxy. No Linux runtime was executed.

**F08-02: Native host launch hardcodes a 1Password extension origin for all hosts.** `src/extensions/nm-proxy.ts:52-66,134-146,189-205` passes `chrome-extension://aeblfdkhhhdcdjpifhhbdiojplfjncoa/` to every native host binary, including non-1Password hosts. The same file assumes a macOS signed relay at `/Applications/Tandem Browser.app/Contents/MacOS/nm-relay`, falling back to direct spawn if absent. This is a source-level host-identity portability gap, not proof of an observed failure.

**F08-03: WSL2 does not inherit Windows native-host registry discovery.** `src/platform/index.ts:64-87` chooses the Linux native-messaging adapter on Linux. Windows registry scanning invokes PowerShell from `src/platform/native-messaging/index.ts:159-225` only through the Windows adapter. `src/extensions/native-messaging.ts:100-145` checks host executable paths with `fs.existsSync`; `src/extensions/nm-proxy.ts:144-147,202-205` spawns the manifest binary path directly. No Windows path conversion, executable migration, or Windows Chrome host discovery is implemented in these paths. Whether a particular WSL interop executable can run remains untested.

**F08-04: Extension install is a separate Tandem-owned copy, not Windows Chrome profile reuse.** `src/extensions/loader.ts:41-48,55-129` reads unpacked extensions from `~/.tandem/extensions` and loads them through Electron `session.extensions.loadExtension`; `src/extensions/crx-downloader.ts:64-105,125-154,173-230` downloads CWS CRX3 and gates it on cryptographic signature verification. This does not automatically import Windows Chrome extension directories, runtime identities, or extension storage. Signature verification is a preservation boundary, not a deletion target.

**F08-05: Startup patches installed extension source on disk.** `src/extensions/manager.ts:152-183` calls action and identity polyfill injection before loading. `src/extensions/action-polyfill.ts:732-775,1021` rewrites MV3 service workers, including legacy-patch stripping; `src/extensions/loader.ts:118-129` invokes `nmProxy.patchManifestCSP`, which may rewrite manifest CSP in `src/extensions/nm-proxy.ts:399-447`. This is actual application design, not a modification performed by this audit. Reproducibility and update/repatch interactions require tests.

**F08-06: Native-message payloads are logged.** `src/extensions/nm-proxy.ts:210-230,243-249` logs up to 300 characters from native-host stdout and 200 characters from WebSocket-to-host JSON, plus host stderr. Payload sensitivity is unknown; credential disclosure was not reproduced. Preserve privacy at the native-host boundary in any future deployment evaluation.

**F08-07: Extension loading is default-session centric.** `src/bootstrap/runtime.ts:420-429` initializes the extension manager for its provided Electron session. `src/extensions/manager.ts:450-500` explicitly documents isolated-session extension loading as not wired into SessionManager. Do not infer named-session extension continuity from default-session startup.

**F08-08: Update checks are scheduled, not auto-applied.** `src/extensions/manager.ts:179-183` starts scheduled checks; `src/extensions/update-checker.ts:675-695` schedules the first after five minutes and subsequent checks by interval; `src/extensions/update-checker.ts:714-736` checks and logs available updates, with no call to `updateOne`/`updateAll` in that scheduled method. CPU/network impact was not measured.

## Ponytail complexity-only findings

- `delete:` `src/extensions/native-messaging.ts:360-421`: unused private `mirrorManifestsToTandemDir()` duplicates the platform adapter's implementation at `src/platform/native-messaging/index.ts:68-113`. Its only occurrence in the inspected class is its declaration; the live call at `native-messaging.ts:172` goes through `detectionAdapter`. Candidate deletion: 53-line method plus imports only if no other references; **not applied**. Estimated net about -55 lines, 0 dependencies.
- `yagni:` `src/extensions/manager.ts:450-500`: `loadInSession` is documented as an unwired future seam. Candidate only, pending a repository-wide caller check and preservation of isolated-session requirements; **not counted** as removable yet.
- Large 1Password-specific polyfill patches are not automatically dead code. Do not delete them without an authenticated-extension acceptance test.

## Tests inspected, not run

`src/extensions/tests/native-messaging.test.ts:11-36` checks a mocked known-host ID alias only. `src/extensions/tests/action-polyfill.test.ts:28-83` simulates injected code in fixtures rather than exercising `injectPolyfills` against a real Electron extension; later assertions mostly inspect source text. `src/platform/tests/platform.test.ts:66-92` covers Linux adapter selection but not WSL2 host invocation. No end-to-end Linux native messaging, extension auth persistence, actual WSLg rendering, GPU, CPU, or memory measurements.

## Evidence-gated next checks (proposals, not changes)

Test Linux manifest discovery versus HTTP/WebSocket proxy lookup with fake harmless host binaries; test per-host origin handling; validate a signed CRX3 extension load and restart without a real authenticated profile; verify redaction of native-message logs; measure extension counts and scheduled update network/CPU cost. Preserve native host allowlist checks, Electron isolation, CRX3 verification, persistent sessions, and the browser automation/Midscene seam.

## Next module

Pass 09: resource cost and optional subsystem inventory (extension overhead, media/voice, UI/theming), selected against the updated cumulative ledger. No implementation changes.
