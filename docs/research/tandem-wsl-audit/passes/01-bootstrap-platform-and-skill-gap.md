# Pass 01: Bootstrap and platform audit

Snapshot: main commit 3b613cfd4c299609ca7ca415d638c1b71c6ba5de.

Read: AGENTS.md, CLAUDE.md, PROJECT.md, ARCHITECTURE.md, package.json, platform source, session manager, startup launcher and tests. Source inspection only; no tests executed.

Verified: Linux platform support is best-effort; Linux platform selection currently uses a macOS user-agent profile and a unit test expects this. The startup script has no explicit WSLg display preflight. SessionManager registers named sessions in process memory while Chromium partitions are persistent.

Next: session lifecycle, recovery, browser profile continuity, cookie persistence, and API routes.
