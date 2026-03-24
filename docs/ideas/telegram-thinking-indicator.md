# Idea: Live Thinking Indicator in Telegram

Show users what the agent is doing while it processes their message — a native typing bubble and/or a live-updating status message.

---

## What Telegram Supports

1. **Typing indicator** — `sendChatAction('typing')` shows the native "..." bubble. Lasts 5 seconds, so needs to repeat every ~4s. NanoClaw already has `setTyping()` in `src/channels/telegram.ts:278` and calls it before/after agent runs (`src/index.ts:240`, `src/index.ts:271`). **Already partially wired — just needs the repeat loop.**

2. **Editable status message** — Send a placeholder message (e.g. "⏳ Thinking..."), save the `message_id`, then edit it in-place with live status as the agent works. Grammy supports `editMessageText` (not currently used). The final reply can either edit the same message or replace it.

---

## What Progress Is Actually Available

| Signal | Available? | Source |
|--------|-----------|--------|
| Agent started thinking | ✅ Yes | `src/index.ts:240` (before runAgent) |
| Agent produced output | ✅ Yes | `onOutput` callback, `src/container-runner.ts:370` |
| Agent done | ✅ Yes | `src/index.ts:271` (after runAgent) |
| Tool use (e.g. "Reading file X") | ❌ Not yet | Needs `agent-runner` change |
| Thinking/reasoning tokens | ❌ Not yet | Needs `agent-runner` change |

The container agent-runner captures tool_use and thinking events internally but doesn't forward them to the host. Adding that would need changes to `container/agent-runner/src/index.ts:392-460`.

---

## Implementation Options

### Option A: Typing Bubble Only (Easy, ~30 min)
Keep repeating `sendChatAction('typing')` every 4s while the agent runs.

**What to change:** `src/index.ts` — wrap the `runAgent()` call with a `setInterval` that fires `setTyping(chatJid, true)` every 4s, clear it on completion.

**Result:** Native Telegram typing bubble the whole time the agent thinks. Simple, zero noise.

---

### Option B: Editable Status Message (Medium, ~2 hrs)
Send "⏳ Thinking..." when processing starts, edit it to "✅ Done" (or just delete it) when the reply arrives.

Could also cycle through messages like:
- `⏳ Thinking...`
- `⏳ Still thinking... (15s)`
- `⏳ Working on it... (30s)`

**What to change:**
- `src/channels/telegram.ts` — add `sendStatusMessage(jid)` and `updateStatusMessage(jid, messageId, text)` using `bot.api.editMessageText`
- `src/index.ts` — call send on start, update/delete on result

---

### Option C: Live Tool Progress (Ambitious, ~half day)
Show exactly what the agent is doing: "🔍 Reading file...", "⚡ Running bash...", "🌐 Fetching URL..."

**Requires agent-runner changes:**
- `container/agent-runner/src/index.ts:392-460` — emit tool_use events as sentinel-wrapped output alongside result events
- `src/container-runner.ts` — parse the new event type
- `src/index.ts` + `src/channels/telegram.ts` — edit the status message with the tool name

---

## Recommendation

Start with **Option A** (typing bubble repeat loop) — it's already half-done and gives instant feedback. Add **Option B** later if you want a visible timer. Option C is the richest UX but requires touching the container agent-runner.
