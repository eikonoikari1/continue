---
name: continue
description: Check latest session's transcript and handoff note, then continue where the previous session left off. Use when resuming work or starting a new session on an ongoing project.
user_invocable: true
---

# Continue Previous Session's Work

Pick up where the last session left off by reading its transcript and any handoff note.

## Step 1 — Find the latest session

Determine the current project's session directory. The project key is derived from `$CWD` by replacing `/` with `-` and prepending `-`. For example, `/Users/eiko/my-project` → `-Users-eiko-my-project`.

Session files live at: `~/.claude/projects/<project-key>/*.jsonl`

Find the **most recently modified `.jsonl` file** (excluding the current session if identifiable). That is the previous session transcript.

## Step 2 — Check for a handoff note

Look for a handoff note in these locations (check all, in order):

1. **Project memory dir** — `~/.claude/projects/<project-key>/memory/handoff*.md` or `*handoff*` files
2. **Project CLAUDE.md** — `~/.claude/projects/<project-key>/CLAUDE.md` — look for a `## Handoff` or `## TODO` or `## Next` section
3. **Repo root** — `$CWD/HANDOFF.md`, `$CWD/.handoff`, or a `## Handoff` section in `$CWD/CLAUDE.md`
4. **End of session transcript** — the last few assistant messages in the JSONL often contain a summary or next-steps

If a handoff note is found, display it and use it as the primary guide for what to do next.

## Step 3 — Parse the session transcript

Read the previous session's JSONL file. Extract:

- **User messages** — what the user was asking/working on
- **Key assistant actions** — file edits, commands run, decisions made
- **Final state** — what was the last thing being worked on, any errors or blockers
- **Unfinished work** — tasks mentioned but not completed, TODOs noted

Build a concise summary (bullet points) of:
1. What was being worked on
2. What was accomplished
3. What remains to do

Keep it short — focus on actionable context, not a full replay.

## Step 4 — Present and continue

Show the user:

```
## Previous Session Summary
- **Project**: <project name/path>
- **Session**: <session id> (<date>)
- **What was done**: <brief summary>
- **What remains**: <remaining items>
- **Handoff note**: <if found, show it>
```

Then ask: **"Ready to continue? Here's what I'd pick up next: [specific next action]. Should I proceed?"**

If `$ARGUMENTS` contains `--auto`, skip the confirmation and start working immediately on the most important remaining item.

## Notes

- The JSONL format has one JSON object per line. Key fields: `type` (user/assistant/system/progress), `message.content` for assistant messages (array of blocks), `content` for user messages.
- Assistant message content blocks can be `text`, `thinking`, `tool_use`, `tool_result` types.
- Focus on `user` and `assistant` type messages with text content. Skip `thinking` blocks and `progress` messages.
- For large sessions, focus on the **last ~50 messages** to get the most recent context rather than parsing the entire file.
