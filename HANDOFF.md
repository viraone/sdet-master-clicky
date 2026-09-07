# Handoff notes for the next agent

Read `README.md` first for the `window.clickyEdit` contract. These are the things the code won't tell you.

## Hard constraints
- **`cs198-analogy.html` is the source of truth.** Never regenerate it (`build_cs198.py` inputs are gone). Edit in place.
- **Do not rename, renumber or remove `data-clicky-id` attributes.** Clicky (`viraone/MyClicky`, separate macOS app) locates boxes on disk by them. Adding boxes is fine → run `python3 scripts/add_clicky_ids.py`; it only assigns to boxes lacking an id, after the current max.
- **Do not change the `window.clickyEdit` method names/shapes or the `#edit=<id>` hash convention** without coordinating with the Clicky app — it is already wired against them. Adding new methods is fine.
- `grep -c 'data-clicky-id' cs198-analogy.html` must equal insight count + code-panel count (currently 88 + 22 = 110). The JS deliberately uses `dataset.clickyId` so the literal string only appears on real boxes.
- No browser-side persistence (localStorage etc.). Persistence is Clicky's job; overlays would mask its on-disk edits. This was removed on purpose.

## How Clicky uses the page (so you can test like it does)
- Clicky runs `do JavaScript` in the tab: edit-verb utterances → `set(id, html)`; questions → `thinking(q)` then `reply(a, {question})`; cancel → `dismiss()`.
- Clicky writes the same edit into the file on disk at **`/Users/viradeth/Desktop/SDET_MASTER_CLICKY/cs198-analogy.html`** and "publish it" pushes to `origin main` of this repo. Don't move the file.
- Editable element: `:scope > p` (insight) or `pre > code` (code panel). Only one box edits at a time.

## Verification recipe (what was used, ~1 min)
`npm i puppeteer-core` in a temp dir, launch `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome` headless, `file://` the page and check: zero console errors; `clickyEdit.list().length === document.querySelectorAll('.insight,.code-panel').length`; click `.clicky-btn.edit` → `location.hash === '#edit=<id>'` → Esc clears it; `set()` returns true and adds `.clicky-flash`; `reply()` inserts `.clicky-reply` after the editing box.

## Known / open
- `index.html` and `code-explainer.html` were carried over untouched; `code-explainer.html` still has its own (unrelated) Gemini chat. Out of scope unless asked.
- Upstream PR `viraone/sdet-master-tracker#18` contains the same commits and is still open; the user decides whether to merge or close it. `upstream` remote points there.
- Tailwind CDN is still loaded (`index.html`-era). Removing it would need a visual pass; preflight currently affects list styling in a few places.

## Typed-edit persistence (September 2026)
- Previously, pencil → type → Done changed only the DOM and was lost on refresh.
- Done / Esc and answer-card Insert now enqueue explicit saves. The matching updated MyClicky app polls `pendingSave`, claims the request, checks its original HTML against disk, writes the box, and publishes only `cs198-analogy.html`.
- The user explicitly authorized publishing these edits to this public GitHub Pages site.
- The bridge accepts only `https://viraone.github.io/sdet-master-clicky/cs198-analogy.html`; Clicky must be running and browser JavaScript from Apple Events enabled. No browser storage was introduced.
- Success has two stages: saved to GitHub, then verified against a fresh fetch of the deployed page. Failures retain dirty state and show retry guidance. Do not label DOM-only changes as saved.
- MyClicky source lives in `/Users/viradeth/Desktop/MyClicky1`; its publish retry now pushes even when the previous attempt already committed.

## Break checkpoint — 2026-09-06
- Site changes are live; Pages build `a9a73ab` was verified HTTP 200 and byte-for-byte against the local lecture.
- MyClicky changes: continuous Ask questions across pauses; retain speech while busy; Ask is the default opening tab; phone TALK respects Ask selection; Done/Esc/Insert save bridge installed and running.
- MyClicky code is committed as `8707d8a` on local main. Pushing main was rejected because GitHub has newer commits; session work is backed up on `codex/ask-stream-and-site-save-20260906`. Integrate with remote main before the next app release; do not force-push.
- Checks passed: release build/signature, JS save-state regression checks, all 110 stable IDs, native disk writes and publish retry against a temporary bare Git repository, exclusion of unrelated staged files.
- In-app browser automation failed to connect, so no end-to-end manual browser edit was claimed. The user should refresh once to load the new controls, keep Clicky running, and wait for “Saved and verified on the live site.”
- Existing untracked `MyClicky_vs_CuaDriver_Strategy.md.pdf` in MyClicky1 was left untouched and not uploaded.
