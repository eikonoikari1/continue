---
name: continue
description: Check latest session's transcript and handoff note, then continue where the previous session left off. Use when resuming work or starting a new session on an ongoing project.
user_invocable: true
---

# Continue Previous Session's Work

Pick up where the last session left off by reading its transcript and any handoff note.

## Step 0 — Check configuration

Read `~/.claude/skills/continue/config.json`. If it does **not** exist, run **First-time setup** (below). If it exists, use its `label_mode` value and skip to Step 1.

If `$ARGUMENTS` contains `--setup`, run First-time setup even if config already exists (reconfigure).

### First-time setup

Ask the user how they want sessions labeled in the picker. Present these options:

1. **first_message** — First user message (what kicked off the session)
2. **last_message** — Last user message (where things ended)
3. **ai_summary** — AI-generated one-line summary (reads first ~10 messages, slower)
4. **combo** — First message as title + last message as subtitle (two lines per entry)
5. **Custom** — Let the user describe their own format

Once the user chooses, write `~/.claude/skills/continue/config.json`:

```json
{
  "label_mode": "first_message",
  "custom_instructions": null
}
```

For custom mode, store the user's description in `custom_instructions`.

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
  - `combo`: first message as title, last message as `└─ last:` subtitle
  - custom: follow `custom_instructions` from config

Present a numbered list:

```
## Recent Sessions
1. [Mar 26 09:48] create a skill (in global dir)
2. [Mar 26 06:46] check latest claude session and continue...
3. [Mar 25 10:38] check handoff note to continue work...
4. [Mar 26 08:44] /continue
5. [Mar 25 13:50] check why /compact skill doesn't work
```

Ask the user which session to continue (number). If `$ARGUMENTS` contains a number (1-5), use that directly without asking.

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

Then ask: **"Ready to continue? Here's what I'd pick up next: [specific next action]. Should I proceed?"**

If `$ARGUMENTS` contains `--auto`, skip the confirmation and start working immediately on the most important remaining item.

## Notes

- The JSONL format has one JSON object per line. Key fields: `type` (user/assistant/system/progress), `message.content` for assistant messages (array of blocks with `role` and `content`), `message.content` for user messages.
- User messages: `type: "user"`, content in `message.content` (string or array). Skip messages with `isMeta: true`.
- Assistant messages: `type: "assistant"`, content in `message.content` (array of blocks — `text`, `thinking`, `tool_use`, `tool_result`). Focus on `text` blocks, skip `thinking`.
- Skip `progress`, `file-history-snapshot`, and `system` type messages when extracting content.
- For large sessions, focus on the **last ~50 messages** to get the most recent context rather than parsing the entire file.
- To reconfigure at any time: `/continue --setup`
