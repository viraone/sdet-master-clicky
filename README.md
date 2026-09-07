# SDET_MASTER_CLICKY

Mobile SDET lecture (`cs198-analogy.html`, the "mall" analogy) as a **read-and-edit** page driven by
[Clicky](https://github.com/viraone/MyClicky), the Mac voice assistant. Spun off from
[`viraone/sdet-master-tracker`](https://github.com/viraone/sdet-master-tracker) (PR #18) so it can evolve independently.

`cs198-analogy.html` **is the source of truth** — edit it in place (the original generator inputs no longer exist).

## How it works
1. Open `cs198-analogy.html` in the browser. Hover any insight card / code panel → toolbar: ＋ ✏️ ⧉ 🗑.
2. Click ✏️ → box enters edit mode, URL becomes `#edit=<data-clicky-id>`.
3. Press TALK on Clicky. Edit-verb utterances → `clickyEdit.set(id, html)`; questions → `clickyEdit.reply(...)` card under the box.
4. Clicky mirrors edits into this file on disk (locating the box by `data-clicky-id`). Say "publish it" to push.

## Stable ids
Every `<div class="insight …">` and `<figure class="code-panel">` carries `data-clicky-id` (`ins-NNN` / `code-NNN`).
Re-run `python3 scripts/add_clicky_ids.py` after adding boxes by hand — idempotent, assigns after the current max.

## `window.clickyEdit` (synchronous, JSON-serialisable, never throw)
| Method | Returns |
|---|---|
| `current()` | `{id, kind: 'insight'\|'code', label, html, text}` for the editing box, or `null` |
| `get(id)` | same shape, or `null` |
| `set(id, html)` | replace editable innerHTML, flash; `true` / `false` |
| `done()` | exit edit mode; `true` |
| `list()` | `[{id, kind, label, text≤120}]` for every box |
| `thinking(question)` | pulsing "Clicky is thinking" card; `true` |
| `reply(answer, {question, id, html})` | render answer (markdown-lite) under the box / floating; `true` |
| `dismiss()` | remove reply card; `true` |

Editable element: `:scope > p` for insights, `pre > code` for code panels. Only one box edits at a time; Esc / ✓ Done exits.
