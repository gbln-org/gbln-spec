# GBLN Tickets System

This directory contains all planning tickets for GBLN implementation.

## Ticket Index

See `ticket-index.csv` for current status of all tickets.

## Ticket Format

Each ticket follows this structure:

```markdown
# Ticket #XXX: Title

**Repo**: repository-name
**Status**: open|in-progress|completed|blocked
**Priority**: critical|high|medium|low
**Created**: YYYY-MM-DD
**Updated**: YYYY-MM-DD

---

## Summary
Brief description of the ticket

## Requirements
Detailed requirements

## Acceptance Criteria
- [ ] Checklist items

## Timeline
Estimated time and dependencies

## Notes
Additional context
```

## Ticket Naming Convention

`XXX-{repo-short-handle}-{topic}.md`

Examples:
- `001-gbln-spec-initial-specification.md`
- `004-gbln-rust-core-parser.md`
- `006-gbln-tools-formatter.md`

## Status Values

- `open`: Not started
- `in-progress`: Currently being worked on
- `completed`: Done and verified
- `blocked`: Waiting on dependencies

## Priority Values

- `critical`: Blocks other work, must be done first
- `high`: Important for initial release
- `medium`: Nice to have for v1.0
- `low`: Future enhancement

## Repository Handles

- `gbln-spec`: Specification repository
- `gbln-rust`: Rust implementation
- `gbln-tools`: CLI tools
- `gbln-python`: Python bindings
- `gbln-js`: JavaScript/TypeScript bindings
- `gbln-website`: Website and playground
- `gbln-vscode`: VSCode extension

## Workflow

1. Create ticket in this directory
2. Add entry to `ticket-index.csv`
3. Update status as work progresses
4. Mark completed when all acceptance criteria met
5. Update `ticket-index.csv` with completion date
