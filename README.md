# /continue — Claude Code Skill

Pick up where your last session left off. This skill reads the previous session's transcript, finds any handoff notes, and offers to continue the work.

## What it does

1. **Finds the latest session** transcript (JSONL) for the current project
2. **Checks for handoff notes** in project memory, CLAUDE.md, repo root, and end of transcript
3. **Parses the transcript** to extract what was worked on, accomplished, and what remains
4. **Presents a summary** and offers to pick up the next action

## Install

```bash
# Clone into your global skills directory
git clone https://github.com/eikonoikari1/claude-skill-continue.git ~/.claude/skills/continue
```

Or copy `SKILL.md` manually into `~/.claude/skills/continue/`.

## Usage

```
/continue          # Show summary and ask before continuing
/continue --auto   # Skip confirmation, start working immediately
```

## Handoff notes

The skill checks these locations for handoff context:

| Location | What it looks for |
|---|---|
| `~/.claude/projects/<key>/memory/` | Files matching `handoff*` |
| `~/.claude/projects/<key>/CLAUDE.md` | `## Handoff`, `## TODO`, or `## Next` sections |
| Repo root | `HANDOFF.md`, `.handoff`, or handoff section in `CLAUDE.md` |
| Session transcript | Last assistant messages with summaries or next-steps |

## Writing a handoff note

At the end of a session, ask Claude to write a handoff note:

> "Write a handoff note summarizing what we did and what's left"

Or add a `## Handoff` section to your project's `CLAUDE.md` manually.

## License

MIT
