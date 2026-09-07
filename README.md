# SDET_MASTER_CLICKY

Mobile SDET lecture (`cs198-analogy.html`, the "mall" analogy) as a **read-and-edit** page driven by
[Clicky](https://github.com/viraone/MyClicky), the Mac voice assistant. Spun off from
[`viraone/sdet-master-tracker`](https://github.com/viraone/sdet-master-tracker) (PR #18) so it can evolve independently.

`cs198-analogy.html` **is the source of truth** — edit it in place (the original generator inputs no longer exist).

## How it works
1. Open `cs198-analogy.html` in the browser. Hover any insight card / code panel → toolbar: ＋ ✏️ ⧉ 🗑.
2. Click ✏️ → box enters edit mode, URL becomes `#edit=<data-clicky-id>`.
3. Press TALK on Clicky. Edit-verb utterances → `clickyEdit.set(id, html)`; questions → `clickyEdit.reply(...)` card under the box.
4. For typed edits on the hosted lecture, **Done / Esc saves through the running Clicky Mac app and publishes to GitHub**. **Insert** on an answer card also saves. Keep the tab open until it says **Saved and verified on the live site**. Clicky needs the browser’s **Allow JavaScript from Apple Events** setting enabled.
5. Voice rewrites still mirror into the source file; say "publish it" to publish them, or finish the edit with Done. Save failures remain visible, and an old tab cannot overwrite a newer version of the same box on disk.

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

### Hosted-page save bridge

The updated Mac app polls the exact hosted lecture URL for explicit save requests.
`pendingSave()` returns `{token, id, html, original}` or `null`; `claimSave(token)` claims one request;
`finishSave(token, ok, message)` acknowledges it. These are additions to the existing synchronous API.
Requests stay in memory only. The page warns before leaving with unsaved changes or an unverified deployment.
Newly inserted boxes are not supported by this save path; add those to the source file through Clicky.
