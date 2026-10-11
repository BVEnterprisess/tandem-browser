# Tandem WSL2 source audit (12 passes)

Repository: BVEnterprisess/tandem-browser. Pinned source commit: `3b613cfd4c299609ca7ca415d638c1b71c6ba5de`. Branch: `research/tandem-wsl-audit`. Read-only implementation audit; no tests or deployment changes.

| Pass | Module | Persistence |
|---|---|---|
| 01 | Bootstrap, platform, canonical layout | Committed: `passes/01-bootstrap-platform-and-skill-gap.md` |
| 02 | Named sessions and restart | Investigated; report not committed |
| 03 | Electron startup/shutdown | Committed: `passes/03-electron-lifecycle-linux-startup.md` |
| 04 | WSLg/display/GPU | Investigated; report not committed |
| 05 | API authorization/network | Local report only |
| 06 | Automation/CDP/Midscene | Local report only |
| 07 | Chrome profile import | Local report only |
| 08 | Extensions/native messaging | Committed: `passes/08-extensions-native-messaging-wsl2.md` |
| 09 | Media, theming, optional subsystems | Local report only |
| 10 | Idle schedulers, watchers, tab/render lifetime | Source research complete; GitHub report write blocked; local report only |
| 11 | Packaging and minimal WSL2 deployment | NEXT |
| 12 | Consolidated verified migration plan | Pending |

Proposals: `proposals/matt-pocock-canonicalization.md`. Source evidence and tests are detailed in individual pass reports. No local-only report counts as committed. Installed skill registry was empty; published Ponytail instructions were read and applied, with provenance disclosed. Preserve authenticated browser state, Electron security, automation interfaces and the Midscene integration boundary. No tests or WSLg benchmarks executed. The only allowed write namespace is `docs/research/tandem-wsl-audit/`.
