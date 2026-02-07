# Context History

This directory maintains the development history and context for this project. It enables Claude Code (and other AI coding assistants) to understand the project's evolution and make informed decisions.

## Directory Structure

- `contexts/` — Summary files for completed work
- `transcripts/` — Full session transcripts (verbatim, do not edit)
- `plans/` — Implementation plans and designs
- `snapshots/` — Point-in-time project state snapshots
- `handoffs/` — Session handoff documents

## Core Files

- `context_index.md` — Master index of all entries (read this first)
- `decisions.md` — Append-only architectural decision log
- `prune_queue.md` — Temporary holding area for content needing consolidation
- `install_log.md` — System-level installation tracking

## Usage with Claude Code

When starting a new session:

1. **Quick orientation**: Read `context_index.md` for an overview
2. **Find specific context**: Use `grep` to search for topics
3. **Understand decisions**: Check `decisions.md` for architectural choices
4. **Review past work**: Read relevant files in `contexts/`
5. **Check current state**: Read the latest snapshot in `snapshots/`

Example queries:
```bash
# Find all context about a topic
grep -r "keyword" contexts/

# See the latest project state
ls -t snapshots/ | head -1

# Find implementation details
grep -l "feature" contexts/ transcripts/
```

## Maintenance

- Update `context_index.md` when adding new context files
- Create snapshots at significant milestones
- Add entries to `decisions.md` for architectural choices
- Keep transcripts verbatim (do not edit)
