# /continue — Claude Code Skill

Pick up where your last session left off. Lists recent sessions, lets you choose which to continue, reads the transcript and any handoff notes, and offers to resume the work.

## What it does

1. **First-run setup** — choose how sessions are labeled (first message, last message, AI summary, combo, or custom)
2. **Lists 5 latest sessions** for the current project with your chosen label format
3. **Checks for handoff notes** in project memory, CLAUDE.md, repo root, and end of transcript
4. **Parses the transcript** to extract what was worked on, accomplished, and what remains
5. **Presents a summary** and offers to pick up the next action

## Install

```bash
# Clone into your global skills directory
git clone https://github.com/eikonoikari1/claude-skill-continue.git ~/.claude/skills/continue
```

Or copy `SKILL.md` manually into `~/.claude/skills/continue/`.

## Usage

```
/continue              # List sessions, pick one, show summary, ask before continuing
/continue 1            # Continue the most recent session directly
/continue --auto       # Skip confirmation, start working immediately
/continue --setup      # Reconfigure label format
```

## Configuration

On first run, you'll be asked how to label sessions in the picker:

| Mode | Description |
|---|---|
| `first_message` | First user message — what started the session |
| `last_message` | Last user message — where things ended |
| `ai_summary` | AI-generated one-liner (slower, reads first ~10 messages) |
| `combo` | First message + last message subtitle |
| Custom | Your own format description |

Config is stored in `~/.claude/skills/continue/config.json` and is **not** read unless the skill is invoked. Reconfigure anytime with `/continue --setup`.

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
