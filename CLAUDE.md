# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single client-facing status report for the **HILL** project (hill.uz — a trilingual ru/uz/en magazine built by Vacuum Agency), published as a static page via GitHub Pages (`.nojekyll`, remote `mdavlatov/hill-status`). It contains no application code: `index.html` *is* the deliverable.

The audience is the client, not developers. Everything is in Russian, written to be read by a non-technical reader — findings are phrased as "what this means for you", not as commit summaries.

## Commands

There is no build, test, or lint step — the page is hand-written HTML opened directly.

```bash
open index.html                    # preview (macOS); also works over GitHub Pages
git log --oneline                  # each revision of the report is one commit
```

After editing, verify tag balance — the file is one long document and a dropped `</div>` is easy to miss and invisible until the layout collapses:

```bash
python3 - <<'EOF'
import io,re
s=io.open('index.html',encoding='utf-8').read()
for t in ['section','div','li','ol','ul','p']:
    o=len(re.findall(r'<%s[\s>]'%t,s)); c=len(re.findall(r'</%s>'%t,s))
    print(t,o,c,'OK' if o==c else 'MISMATCH')
EOF
```

To proofread a section as plain text without wading through markup:

```bash
python3 -c "
import io,re
s=io.open('index.html',encoding='utf-8').read()
m=re.search(r'<section id=\"summary\".*?</section>',s,re.S)
print(re.sub(r'<[^>]+>','',m.group(0)))"
```

## File structure gotcha

`index.html` is ~550 lines but ~545 KB: six screenshots inside §06 are embedded as base64 `data:image/jpeg` URIs, and those lines run to 130 KB each. Find them before reading anything nearby — the line numbers shift with every edit:

```bash
awk '{print NR": "length($0)}' index.html | sort -t: -k2 -rn | head -8
```

**Never `cat`, `Read`, or `sed -n` a range that includes them** — it floods the context. Use `cut -c1-400` to truncate, target line ranges that exclude them, or extract sections with the Python snippets above. Prefer exact-string replacement (a Python script asserting `s.count(old)==1`) over line-addressed `sed` for edits.

## Where the report's content comes from

This repo holds no source data. Every fact in the report is pulled from three places, and a "update the report" task means re-reading all three:

| Source | What it provides |
| --- | --- |
| `Vacuum-Agency/hill-fix` | The task tracker. `README.md` holds the bug/finalization/"новые правки" tables with 🔴 blocking / 🟡 non-blocking / 🔵 discuss categories, estimates and done-strikethroughs. `CONTENT-STATUS.md` is a CMS snapshot (article counts per rubric, what's missing, which sections are hidden). |
| `Vacuum-Agency/hill-website` | The actual site — Astro + Sanity CMS. Commit log shows what shipped since the last report; `src/lib/sanity.ts` carries the `isDisabled` menu flags that determine which sections are still hidden. |
| hill.uz (live) | Ground truth. `curl` the live pages to confirm claims before writing them — HTTP status of hidden sections, `<title>`, favicon tags, footer links, sitemap counts. |

The tracker goes stale: rows stay `open` after the work shipped. When the tracker and the live site disagree, verify against the live site and flag the discrepancy to the user rather than silently picking one.

## Report structure

Nine numbered sections, each `<section id="…">` with a `.kicker` block carrying its number:

`01 summary` (prose "коротко о главном") · `02 glance` (four `.stat` tiles) · `03 done` · `04 extra` (work done outside the contract) · `05 pages` (live-URL table) · `06 fidelity` (Figma-vs-implementation, incl. the embedded screenshot pairs) · `07 remaining` (our-side task cards) · `08 ask` (numbered client asks) · `09 launch` (next steps).

`02`, `07` and `08` carry the numbers that must stay consistent with each other: the tile counts, the card totals, and the ask-list chip (`N пунктов · ждём вас`) all restate the same figures.

## Design system

Editing means reusing these classes, never inventing new CSS. Colours come from CSS custom properties defined four times over (`:root`, `prefers-color-scheme: dark`, and both `[data-theme]` overrides for the fixed toggle button) — a new colour must be added to all of them or dark mode breaks.

- **Status chips**: `.chip.done` (green), `.chip.prog` (amber), `.chip.act` (red — needs the client). Same triple drives `--done` / `--prog` / `--act`.
- **`.find`** blocks in §06: `.find.ok` / `.find.work` / `.find.attn` set the left border colour to match.
- **`.card`** lists in §07: `li.is-done` strikes the label and greens the `.est`; `.sum` in the card `h3` is the right-aligned total.
- **`.asklist`** in §08 is auto-numbered via CSS counters; `li.is-done` swaps the number for a ✓ and skips the counter, so **striking an item through renumbers everything below it**.
- `.stat` tiles, `.feat` icon lists, `.steps` (with `li.now` marking the current stage), `.mono`, `.serif`, `.tnum`.

## Editing conventions

- **Completed items are struck through, never deleted** — `<s>` in prose, `class="is-done"` in lists. The report's value is that the client can see the history of what was closed.
- **Update content only.** Do not restyle, reformat, or reorganize unless asked. Keep new inline `style="…"` limited to patterns already present in the file.
- **The report date appears in three places** and all three must move together: the `.metaline` span in the masthead, the `.foot` span at the bottom, and the "Подготовлено" line in `README.md`.
- New client asks are **appended** to the end of the §08 list rather than inserted, so item numbers stay stable between revisions.
- Don't state a fact you haven't verified against a source above. Estimates carried over from the tracker keep the tracker's own wording (`~45м`, `≈ 9ч 15м`).

## Commits

One commit per report revision, subject-only, Russian:

```
docs: отчёт от 31 августа — техчасть закрыта, список правок вырос до 14 пунктов
```

Also seen for narrower edits: `docs: §08 — …`, `docs: синхронизация с hill-fix — …`.
