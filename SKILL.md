---
name: continue
description: Check latest session's transcript and handoff note, then continue where the previous session left off. Use when resuming work or starting a new session on an ongoing project.
user_invocable: true
---

# Continue Previous Session's Work

Pick up where the last session left off by reading its transcript and any handoff note.

**IMPORTANT**: This skill uses the `AskUserQuestion` tool for all user interactions — setup, session picking, and confirmation. Never use plain text prompts to ask the user to choose.

## Step 0 — Check configuration

Read `~/.claude/skills/continue/config.json`. If it does **not** exist, run **First-time setup** (below). If it exists, use its `label_mode` value and skip to Step 1.

If `$ARGUMENTS` contains `--setup`, run First-time setup even if config already exists (reconfigure).

### First-time setup

Use `AskUserQuestion` to ask the user how they want sessions labeled:

```
question: "How should sessions be labeled in the picker?"
header: "Label format"
options:
  - label: "First message"
    description: "What kicked off the session, e.g. 'create a skill (in global dir)'"
  - label: "Last message"
    description: "Where things ended, e.g. 'update this skill to list 5 latest...'"
  - label: "AI summary"
    description: "One-line AI-generated summary (slower — reads first ~10 messages)"
  - label: "First + last combo"
    description: "First message as title, last message as subtitle (two lines per entry)"
```

The user can also pick "Other" to describe a custom format.

Once chosen, write `~/.claude/skills/continue/config.json`:

```json
{
  "label_mode": "first_message",
  "custom_instructions": null
}
```

Valid `label_mode` values: `first_message`, `last_message`, `ai_summary`, `combo`. For custom, set `label_mode` to `"custom"` and store the user's description in `custom_instructions`.

Then continue to Step 1.

## Step 1 — List the 5 latest sessions

Determine the current project's session directory. The project key is derived from `$CWD` by replacing `/` with `-` and prepending `-`. For example, `/Users/foo/my-project` → `-Users-foo-my-project`.

Session files live at: `~/.claude/projects/<project-key>/*.jsonl`

Find the **5 most recently modified `.jsonl` files**. For each, extract:

- **Timestamp** — from the first message's `timestamp` field
- **Session ID** — from `sessionId` field
- **Label** — based on `label_mode` from config:
  - `first_message`: first non-meta `user` type message → `message.content` (truncate to 60 chars)
  - `last_message`: last non-meta `user` type message → `message.content` (truncate to 60 chars)
  - `ai_summary`: read first ~10 user/assistant text messages, generate a one-line summary
  - `combo`: use first message as label, append ` → last: <last msg>` in description
  - custom: follow `custom_instructions` from config

### Present via AskUserQuestion

Use `AskUserQuestion` to let the user pick a session. Map each session to an option:

```
question: "Which session do you want to continue?"
header: "Session"
options:
  - label: "[Mar 26 09:48] create a skill (in global dir)"
    description: "Session abc123 · 190KB"
  - label: "[Mar 26 06:46] check latest claude session and..."
    description: "Session def456 · 5.9MB"
  - label: "[Mar 25 10:38] check handoff note to continue..."
    description: "Session ghi789 · 2.7MB"
  - label: "[Mar 25 13:50] check why /compact skill doesn't..."
    description: "Session jkl012 · 314KB"
```

Note: AskUserQuestion supports max 4 options. Show the 4 most recent sessions as options. The user can pick "Other" to specify a session ID or request more.

If `$ARGUMENTS` contains a number (1-4), skip the picker and use that session directly.

## Step 2 — Check for a handoff note

Look for a handoff note in these locations (check all, in order):

1. **Project memory dir** — `~/.claude/projects/<project-key>/memory/handoff*.md` or `*handoff*` files
2. **Project CLAUDE.md** — `~/.claude/projects/<project-key>/CLAUDE.md` — look for a `## Handoff` or `## TODO` or `## Next` section
3. **Repo root** — `$CWD/HANDOFF.md`, `$CWD/.handoff`, or a `## Handoff` section in `$CWD/CLAUDE.md`
4. **End of session transcript** — the last few assistant messages in the JSONL often contain a summary or next-steps

If a handoff note is found, display it and use it as the primary guide for what to do next.

## Step 3 — Parse the selected session transcript

Read the selected session's JSONL file. Extract:

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
## Session Summary
- **Project**: <project name/path>
- **Session**: <session id> (<date>)
- **What was done**: <brief summary>
- **What remains**: <remaining items>
- **Handoff note**: <if found, show it>
```

Then use `AskUserQuestion` to confirm next steps:

```
question: "How should I continue?"
header: "Next step"
options:
  - label: "<specific next action inferred from session>"
    description: "Pick up the most important remaining item"
  - label: "Show full transcript context"
    description: "Display more detail before deciding"
  - label: "Do nothing"
    description: "Just wanted the summary, don't continue any work"
```

If `$ARGUMENTS` contains `--auto`, skip this confirmation and start working immediately on the most important remaining item.

## Notes

- The JSONL format has one JSON object per line. Key fields: `type` (user/assistant/system/progress), `message.content` for assistant messages (array of blocks with `role` and `content`), `message.content` for user messages.
- User messages: `type: "user"`, content in `message.content` (string or array). Skip messages with `isMeta: true`.
- Assistant messages: `type: "assistant"`, content in `message.content` (array of blocks — `text`, `thinking`, `tool_use`, `tool_result`). Focus on `text` blocks, skip `thinking`.
- Skip `progress`, `file-history-snapshot`, and `system` type messages when extracting content.
- For large sessions, focus on the **last ~50 messages** to get the most recent context rather than parsing the entire file.
- To reconfigure at any time: `/continue --setup`
