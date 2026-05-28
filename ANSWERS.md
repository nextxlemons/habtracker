# ANSWERS.md

## 1. How to run

Zero dependencies — this is a single HTML file.

```bash
# Simplest: open directly
open index.html

# Or serve locally
python3 -m http.server 8080
# → http://localhost:8080

# Or with Node
npx serve .
```

No npm install, no build step. Works in any modern browser.

---

## 2. Stack & design choices

**Stack:** Vanilla HTML, CSS, and JavaScript. No framework, no bundler.

For a self-contained assessment deliverable that needs to "run in a browser" with a single command, vanilla is the clearest choice — no build chain to break, nothing to install, nothing to misconfigure on a reviewer's machine. A React SPA would have added complexity with zero functional upside here; the UI is simple enough that DOM manipulation is readable and fast.

**Design decision 1 — The grid dominates the viewport (not the form)**

The add-habit input is small and secondary; the grid takes up the bulk of the vertical space. This is intentional: a habit tracker's primary job is *showing you where you stand*, not *adding new habits* (which is a rare action). In most trackers I've seen, the input competes visually with the data — I wanted the grid to be the thing your eye goes to first. The habit name column is `1fr` (fills available space) while day columns are fixed-width (`52px` on desktop), so the grid reads left-to-right naturally and the name column gets what it needs regardless of screen width.

**Design decision 2 — The streak badge uses tiered color, not just a number**

A streak of 1 is grey (`--text-dim`). At 3+ days it goes amber (`--accent-3`). At 7+ days it goes green (`--accent`) and adds a 🔥 emoji. This means you can glance at the streak column and immediately understand who's "in the zone" without reading a single number. A flat number with no visual weight communicates nothing at a glance. The three tiers (cold / warm / hot) are visually distinct but use the same monospaced font so the column stays optically stable.

**Week starts on Monday.** Monday is the ISO 8601 standard weekday, the cognitive start of the work/routine week for most people, and it makes Sun–Sat weeks feel natural on a Mon–Sun grid. A Sunday-start grid would put the weekend in the middle visually, which feels off for habit-building (habits live in the rhythm of the weekday, not the weekend calendar).

---

## 3. Responsive & accessibility

**Responsive behavior:**

At `360px` (narrow phone): the grid column width shrinks to `42px`, the shell padding tightens to `12px`, and the header stacks vertically (wordmark above week nav). The grid itself is wrapped in a horizontally-scrollable container (`.grid-scroll`) so the 7-column grid never reflows into something unreadable — it scrolls naturally. The summary bar stacks to a column. The habit name column is `1fr` so short names stay legible.

At `1440px` (wide laptop): column width expands to `60px`, the name column fills comfortably, and the summary bar sits in a single row. The max-width cap of `900px` keeps lines from getting too wide and the tracker from sprawling into a spreadsheet feel.

**Accessibility handled:**

- Every interactive element (`check-btn`, `nav-btn`, `add-btn`, `edit-btn`, `delete-btn`) has an explicit `aria-label` describing its action *and* which habit/date it refers to (e.g. "Check Exercise on May 28"). This means screen reader users don't hear 49 identical "button" announcements.
- Check buttons use `aria-pressed` (true/false) — the correct ARIA pattern for toggles, not checkboxes.
- The summary bar has `aria-live="polite"` so completions update are announced without interrupting.
- Focus states: every interactive element has a visible `focus-visible` outline (2px solid `--accent` with offset), which I did not suppress — a common mistake.
- The progress bar uses `role="progressbar"` with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`.

**Accessibility knowingly skipped:**

I did not add full keyboard navigation across the grid cells (arrow keys to move between checkboxes). Implementing a full ARIA `grid` with roving tabindex would roughly double the JS complexity and is a significant undertaking to do correctly. For a working prototype, tab-order through the cells is sufficient for screen reader use. In a production build I'd implement the `grid` keyboard interaction pattern from ARIA Authoring Practices.

---

## 4. AI usage

I used Claude (claude.ai) for this project in the following places:

**1. Initial structure scaffold**
I asked for a semantic HTML structure for a "weekly habit grid with a name column and 7 day columns." It gave me a table-based layout. I switched it to CSS Grid because `<table>` requires explicit `thead/tbody/tr/td` nesting that becomes awkward when rows are dynamically managed via JS — with CSS Grid I can build each row as a `div` with `display: grid` inheriting the parent's column template, which is much simpler to generate and animate.

**2. Streak calculation logic**
I asked for a function to "calculate the current streak for a habit from a set of date keys." The AI gave me a forward-scan loop starting from the habit's creation date. I rewrote it as a backwards scan from today (or yesterday, if today is unchecked) because: (a) it's O(streak length) instead of O(all days since creation), and (b) the "start from today and go back" mental model matches how users think about streaks.

**3. CSS animation for check toggle**
I asked for "a satisfying CSS animation when a checkbox is checked." It gave me a `transform: scale` bounce on the button itself. I kept the bounce but added a `::after` pseudo-element that expands and fades — the "ripple" effect — because the button bounce alone felt too subtle. The ripple gives spatial feedback (you can see *where* you clicked) without being distracting.
**4. To write ANSWERS.md and README.md**
I have use Claude to write this both files.


---

## 5. Honest gap

The streak calculation has a known limitation: it only looks at localStorage data, so if you've been checking habits for months, there's no way to backfill or import historical data. More concretely — if you used this app for two weeks, cleared your browser storage, and restarted, your streaks reset to 0 with no recovery path.

With another day I'd add a simple data export/import: a "Download backup (JSON)" button and a "Restore from backup" file input. The data model (`habits` array + `checks` flat object) is already JSON-serializable, so this would be maybe 30 lines of code. I'd also add a "delete with confirm" dialog instead of instant deletion, since there's currently no undo for accidentally removed habits.