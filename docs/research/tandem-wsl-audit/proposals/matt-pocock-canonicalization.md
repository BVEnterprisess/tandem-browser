# Canonicalization proposals (not applied)

These are ready-to-review drafts. No existing root file or implementation is changed.

## Agent instruction pointer for CLAUDE.md

The inspected setup-matt-pocock-skills source selects CLAUDE.md when that file exists. Preserve its current AGENTS.md pointer and append:

```md
## Agent skills

### Issue tracker
Engineering work is tracked in GitHub Issues. See docs/agents/issue-tracker.md.

### Domain docs
Use single-context vocabulary in GLOSSARY.md when terms are resolved; read relevant docs/adr/ decisions. See docs/agents/domain.md.
```

Do not add a triage-label block until the triage skill and existing labels are confirmed.

## Candidate docs/agents/issue-tracker.md

```md
# Issue tracker
Work items and decision tickets live in GitHub Issues for BVEnterprisess/tandem-browser.
Use native dependencies and sub-issues when available.
Record decisions with evidence links; preserve issue identity.
PRs are not a triage request surface by default.
```

## Candidate docs/agents/domain.md

```md
# Domain docs
Single-context layout. Read GLOSSARY.md if present and relevant docs/adr/ before changes.
Create the glossary lazily when domain terms are agreed. Define vocabulary, not implementation.
Record ADRs only for hard-to-reverse, surprising, genuine trade-offs.
```

## Namespace reconciliation

AGENTS.md and package.json contain upstream hydro13 references while the audit target is BVEnterprisess/tandem-browser. Separate fork-specific repository metadata from upstream attribution. Do not redirect release publishing without a separate distribution decision. A root CONTEXT.md would duplicate existing PROJECT.md, ARCHITECTURE.md, and AGENTS.md unless a distinct need is established. No root files are touched.
