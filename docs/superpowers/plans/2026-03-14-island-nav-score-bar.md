# Pill Island Navigation & Score Bar Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Combine top-nav + per-page filter rows into an animated pill island, and move score bar inside the flashcard as a top strip.

**Architecture:** Replace `.top-nav`, `.filter-row`, `.content-nav`, `.notes-filter-row` with a single `.nav-island` component. Island has tabs on top and per-page sub-filters that slide open/close via CSS grid animation. Score bar moves from standalone div into `.card-front` as a top strip.

**Tech Stack:** Vanilla HTML/CSS/JS in a single `index.html` file. No frameworks, no build tools.

**Spec:** `docs/superpowers/specs/2026-03-14-desktop-layout-island-nav-design.md`

---

## Chunk 1: Pill Island Navigation

### Task 1: Add Nav Island CSS

**Files:**
- Modify: `index.html` — CSS section, replace `.top-nav` and `.filter-row` styles (~lines 533-570, 867-898, 955-978, 1122-1134)

- [ ] **Step 1: Add `.nav-island` CSS after the existing `.score-bar` styles (line ~542)**

Add these new styles right before the `/* ===== CARD ===== */` comment (line 572):

```css
/* ===== NAV ISLAND ===== */
.nav-island {
  display: inline-flex;
  flex-direction: column;
  align-items: center;
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: 24px;
  padding: 4px;
  box-shadow: var(--shadow);
  transition: border-radius 0.3s;
  margin-bottom: clamp(8px, 1.5vh, 16px);
}

.nav-island.expanded {
  border-radius: 20px;
}

.nav-island-tabs {
  display: flex;
  gap: 4px;
  width: 100%;
}

.nav-island-tab {
  flex: 1;
  padding: 8px 20px;
  border: none;
  background: transparent;
  border-radius: 20px;
  font-family: 'Inter', sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: var(--text-muted);
  cursor: pointer;
  transition: all 0.2s;
}

.nav-island-tab.active {
  background: var(--btn-dark-bg);
  color: var(--card-back-text);
}

.nav-island-filters {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.3s ease;
  width: 100%;
}

.nav-island.expanded .nav-island-filters {
  grid-template-rows: 1fr;
}

.nav-island-filters-inner {
  overflow: hidden;
  min-height: 0;
}

.nav-island-filter-group {
  display: none;
  flex-wrap: wrap;
  gap: 4px;
  padding: 6px 4px 4px;
  justify-content: center;
}

.nav-island-filter-group.active {
  display: flex;
}
```

- [ ] **Step 2: Add mobile overrides for `.nav-island` in existing media queries**

In `@media (max-width: 520px)` block (line ~1259), add:

```css
.nav-island { width: 100%; border-radius: 16px; }
.nav-island.expanded { border-radius: 14px; }
.nav-island-tab { padding: 10px 12px; min-height: 44px; font-size: 12px; }
.nav-island-filter-group { gap: 6px; }
.nav-island-filter-group .filter-btn { padding: 8px 12px; min-height: 40px; font-size: 12px; }
.nav-island-filter-group .content-nav-btn { padding: 10px 14px; min-height: 44px; font-size: 13px; }
.nav-island-filter-group .notes-filter-btn { padding: 10px 14px; min-height: 44px; }
```

- [ ] **Step 3: Verify CSS syntax is valid**

Open `index.html` in browser, check DevTools console for CSS parsing errors.

### Task 2: Replace HTML — Top Nav + Filters → Nav Island

**Files:**
- Modify: `index.html` — HTML section (~lines 1387-1405, 1450-1455, 2421-2425)

- [ ] **Step 1: Replace `.top-nav` div (lines 1387-1391) with nav island HTML**

Replace:
```html
<div class="top-nav">
  <button class="top-nav-btn active" data-page="words" id="navWords">Kelimeler</button>
  <button class="top-nav-btn" data-page="content" id="navContent">Konular</button>
  <button class="top-nav-btn" data-page="notes" id="navNotes">Notlar</button>
</div>
```

With:
```html
<div class="nav-island" id="navIsland">
  <div class="nav-island-tabs">
    <button class="nav-island-tab active" data-page="words" id="navWords">Kelimeler</button>
    <button class="nav-island-tab" data-page="content" id="navContent">Konular</button>
    <button class="nav-island-tab" data-page="notes" id="navNotes">Notlar</button>
  </div>
  <div class="nav-island-filters">
    <div class="nav-island-filters-inner">
      <div class="nav-island-filter-group active" data-filters-for="words">
        <button class="filter-btn active" data-filter="all">Hepsi</button>
        <button class="filter-btn" data-filter="favs">Favoriler</button>
        <button class="filter-btn" id="quizStartBtn" style="background:var(--accent);color:#fff;border-color:var(--accent)">Quiz</button>
        <button class="filter-btn" id="matchStartBtn">E&#351;le&#351;tir</button>
        <button class="filter-btn" id="gramStartBtn" style="background:var(--btn-dark-bg);color:var(--card-back-text);border-color:var(--btn-dark-bg)">Grammar</button>
      </div>
      <div class="nav-island-filter-group" data-filters-for="content">
        <button class="content-nav-btn active" data-unit="1">Unit 1</button>
        <button class="content-nav-btn" data-unit="2">Unit 2</button>
        <button class="content-nav-btn" data-unit="3">Unit 3</button>
      </div>
      <div class="nav-island-filter-group" data-filters-for="notes">
        <button class="notes-filter-btn active" data-notes-filter="all">T&#252;m&#252;</button>
        <button class="notes-filter-btn" data-notes-filter="standalone">Genel</button>
        <button class="notes-filter-btn" data-notes-filter="attached">Konulara Ba&#287;l&#305;</button>
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: Remove the old `.filter-row` div from `#wordsPage` (lines 1399-1405)**

Delete:
```html
<div class="filter-row">
  <button class="filter-btn active" data-filter="all">Hepsi</button>
  <button class="filter-btn" data-filter="favs">Favoriler</button>
  <button class="filter-btn" id="quizStartBtn" style="background:var(--accent);color:#fff;border-color:var(--accent)">Quiz</button>
  <button class="filter-btn" id="matchStartBtn">E&#351;le&#351;tir</button>
  <button class="filter-btn" id="gramStartBtn" style="background:var(--btn-dark-bg);color:var(--card-back-text);border-color:var(--btn-dark-bg)">Grammar</button>
</div>
```

- [ ] **Step 3: Remove the old `.content-nav` div from `#contentPage` (lines 1451-1455)**

Delete:
```html
<div class="content-nav">
  <button class="content-nav-btn active" data-unit="1">Unit 1</button>
  <button class="content-nav-btn" data-unit="2">Unit 2</button>
  <button class="content-nav-btn" data-unit="3">Unit 3</button>
</div>
```

- [ ] **Step 4: Remove the old `.notes-filter-row` from `#notesPage` (lines 2421-2425)**

Delete:
```html
<div class="notes-filter-row">
  <button class="notes-filter-btn active" data-notes-filter="all">T&#252;m&#252;</button>
  <button class="notes-filter-btn" data-notes-filter="standalone">Genel</button>
  <button class="notes-filter-btn" data-notes-filter="attached">Konulara Ba&#287;l&#305;</button>
</div>
```

- [ ] **Step 5: Verify HTML renders correctly**

Open in browser: all 3 tabs visible in pill island, no broken layout.

### Task 3: Update JavaScript — Island Toggle + Page Switch

**Files:**
- Modify: `index.html` — JS section (~lines 3410-3436)

- [ ] **Step 1: Replace `switchPage` function and tab event listeners (lines 3410-3426)**

Replace:
```javascript
var navWords = el('navWords');
var navContent = el('navContent');

function switchPage(page) {
  currentPage = page;
  wordsPage.style.display = page === 'words' ? '' : 'none';
  contentPage.classList.toggle('active', page === 'content');
  notesPage.classList.toggle('active', page === 'notes');
  navWords.classList.toggle('active', page === 'words');
  navContent.classList.toggle('active', page === 'content');
  navNotes.classList.toggle('active', page === 'notes');
  if (page === 'notes') renderNotesPage();
}

navWords.addEventListener('click', function() { switchPage('words'); });
navContent.addEventListener('click', function() { switchPage('content'); });
navNotes.addEventListener('click', function() { switchPage('notes'); });
```

With:
```javascript
var navWords = el('navWords');
var navContent = el('navContent');
var navIsland = el('navIsland');
var islandExpanded = false;

function switchPage(page) {
  currentPage = page;
  wordsPage.style.display = page === 'words' ? '' : 'none';
  contentPage.classList.toggle('active', page === 'content');
  notesPage.classList.toggle('active', page === 'notes');
  document.querySelectorAll('.nav-island-tab').forEach(function(t) {
    t.classList.toggle('active', t.dataset.page === page);
  });
  document.querySelectorAll('.nav-island-filter-group').forEach(function(g) {
    g.classList.toggle('active', g.dataset.filtersFor === page);
  });
  if (page === 'notes') renderNotesPage();
}

function toggleIsland(page) {
  if (currentPage === page) {
    islandExpanded = !islandExpanded;
  } else {
    switchPage(page);
    islandExpanded = true;
  }
  navIsland.classList.toggle('expanded', islandExpanded);
}

navWords.addEventListener('click', function() { toggleIsland('words'); });
navContent.addEventListener('click', function() { toggleIsland('content'); });
navNotes.addEventListener('click', function() { toggleIsland('notes'); });
```

- [ ] **Step 2: Verify existing content-nav-btn listeners still work (lines 3428-3436)**

The existing `document.querySelectorAll('.content-nav-btn')` listener binds by class, so it still works because buttons keep their class. No changes needed here.

- [ ] **Step 3: Verify existing filter-btn listeners still work (line ~2796)**

The existing `document.querySelectorAll('.filter-btn')` listener binds by class. Buttons keep their class. No changes needed.

- [ ] **Step 4: Verify existing notes-filter-btn listeners still work (line ~3369)**

Same pattern — class-based binding, no changes needed.

- [ ] **Step 5: Test in browser**

1. Click "Kelimeler" — island expands, shows Hepsi/Favoriler/Quiz/Eslestir/Grammar
2. Click "Kelimeler" again — island collapses
3. Click "Konular" — page switches, island expands, shows Unit 1/Unit 2/Unit 3
4. Click "Notlar" — page switches, shows Tumu/Genel/Konulara Bagli
5. Click "Notlar" again — island collapses
6. Verify Unit buttons switch content, filter buttons filter words, notes filter buttons filter notes

### Task 4: Clean Up Old CSS

**Files:**
- Modify: `index.html` — CSS section

- [ ] **Step 1: Remove old `.top-nav` styles (lines ~867-898)**

Delete the `/* ===== TOP NAV ===== */` block:
```css
/* ===== TOP NAV ===== */
.top-nav { ... }
.top-nav-btn { ... }
.top-nav-btn:first-child { ... }
.top-nav-btn:last-child { ... }
.top-nav-btn.active { ... }
```

- [ ] **Step 2: Remove old `.top-nav-btn:not(:first-child)` fix (line ~1123)**

Delete:
```css
/* ===== TOP NAV 3-BUTTON FIX ===== */
.top-nav-btn:not(:first-child):not(:last-child) { border-radius: 0; border-right: none; }
```

- [ ] **Step 3: Remove old `.filter-row` max-width style (lines ~544-551)**

Delete:
```css
.filter-row {
  display: flex;
  gap: 8px;
  margin-bottom: 8px;
  flex-wrap: wrap;
  width: 100%;
  max-width: clamp(300px, 90%, 1100px);
}
```

- [ ] **Step 4: Remove old `.content-nav` and `.content-nav-btn` base styles (lines ~955-978)**

Note: Keep `.content-nav-btn` and `.content-nav-btn.active` styles since the buttons still use those classes inside the island.

Only delete:
```css
.content-nav {
  display: flex;
  gap: 8px;
  margin-bottom: 24px;
  flex-wrap: wrap;
}
```

- [ ] **Step 5: Remove old `.notes-filter-row` style (line ~1132)**

Delete:
```css
.notes-filter-row { display: flex; gap: 8px; margin-bottom: 16px; flex-wrap: wrap; }
```

- [ ] **Step 6: Update mobile breakpoint references**

In `@media (max-width: 520px)`:
- Remove: `.filter-row { gap: 6px; justify-content: center; }` (line ~1281)
- Remove: `.top-nav { padding: 8px 4px 0; }` (line ~1283)
- Remove: `.top-nav-btn { padding: 10px 12px; min-height: 44px; font-size: 12px; }` (line ~1284)
- Remove: `.content-nav { gap: 6px; }` (line ~1285)
- Keep `.filter-btn`, `.content-nav-btn`, `.notes-filter-btn` mobile overrides (they still apply inside island)

- [ ] **Step 7: Verify nothing is broken**

Open in browser, test all 3 pages and all 5 themes. Check mobile responsive view.

- [ ] **Step 8: Commit**

```bash
git add index.html
git commit -m "feat: replace top-nav and filter rows with animated pill island navigation"
```

---

## Chunk 2: Score Bar Inside Card

### Task 5: Move Score Bar Into Card Front

**Files:**
- Modify: `index.html` — CSS section (~line 533), HTML section (~lines 1394-1397, 1410-1418)

- [ ] **Step 1: Update `.score-bar` CSS (lines ~533-542)**

Replace:
```css
.score-bar {
  display: flex;
  gap: 16px;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 4px;
}

.score-correct { color: var(--correct); }
.score-wrong { color: var(--wrong); }
```

With:
```css
.score-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  padding: 8px 16px;
  background: var(--tag-bg);
  border-bottom: 1px solid var(--card-border);
  font-size: 12px;
  font-weight: 600;
  box-sizing: border-box;
  flex-shrink: 0;
}

.score-bar-left {
  display: flex;
  gap: 12px;
}

.score-correct { color: var(--correct); }
.score-wrong { color: var(--wrong); }
.score-progress { color: var(--text-muted); font-weight: 500; }
```

- [ ] **Step 2: Update `.card-front` layout to flex-column (line ~622)**

Replace:
```css
.card-front {
  background: var(--card-bg);
  border: 1px solid var(--card-border);
}
```

With:
```css
.card-front {
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  justify-content: flex-start;
}

.card-front-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 0 32px;
  width: 100%;
  box-sizing: border-box;
}
```

Note: `.card-face` already has `flex-direction: column`. By changing `.card-front` to `justify-content: flex-start`, the score bar sits at top naturally, and the new `.card-front-content` wrapper centers the word content in the remaining space.

- [ ] **Step 3: Move score bar HTML from `#wordsPage` into `.card-front` and wrap content**

Remove score bar from its current location (lines 1394-1397):
```html
<div class="score-bar">
  <span class="score-correct" id="scoreCorrect">0</span>
  <span class="score-wrong" id="scoreWrong">0</span>
</div>
```

Restructure `.card-front` (lines 1410-1418) from:
```html
<div class="card-face card-front">
  <button class="tts-btn" id="ttsBtn" title="Sesli oku">&#128264;</button>
  <button class="fav-btn" id="favBtn">&#9825;</button>
  <div class="word" id="wordText">Loading...</div>
  <div class="pos" id="posText"></div>
  <div class="pronunciation" id="pronText"></div>
  <div class="section-tag" id="sectionText"></div>
  <div class="tap-hint">kart&#305; &#231;evirmek i&#231;in t&#305;kla</div>
</div>
```

To:
```html
<div class="card-face card-front">
  <div class="score-bar">
    <div class="score-bar-left">
      <span class="score-correct" id="scoreCorrect">&#10003; 0</span>
      <span class="score-wrong" id="scoreWrong">&#10007; 0</span>
    </div>
    <span class="score-progress" id="scoreProgress">0 / 0</span>
  </div>
  <div class="card-front-content">
    <button class="tts-btn" id="ttsBtn" title="Sesli oku">&#128264;</button>
    <button class="fav-btn" id="favBtn">&#9825;</button>
    <div class="word" id="wordText">Loading...</div>
    <div class="pos" id="posText"></div>
    <div class="pronunciation" id="pronText"></div>
    <div class="section-tag" id="sectionText"></div>
    <div class="tap-hint">kart&#305; &#231;evirmek i&#231;in t&#305;kla</div>
  </div>
</div>
```

- [ ] **Step 4: Add `scoreProgress` variable and update all score JS locations**

Add variable declaration at line ~2491 (after existing `scoreWrong`):
```javascript
var scoreProgress = el('scoreProgress');
```

Update score reset in `applyFilter()` (line ~2658):
Replace:
```javascript
scoreCorrect.textContent = '0';
scoreWrong.textContent = '0';
```
With:
```javascript
scoreCorrect.textContent = '✓ 0';
scoreWrong.textContent = '✗ 0';
```

Update score display in `checkAnswer()` area (line ~2748):
Replace:
```javascript
scoreCorrect.textContent = correctCount;
scoreWrong.textContent = wrongCount;
```
With:
```javascript
scoreCorrect.textContent = '✓ ' + correctCount;
scoreWrong.textContent = '✗ ' + wrongCount;
```

Update score reset in sidebar file-filter listener (line ~2791):
Replace:
```javascript
scoreCorrect.textContent = '0';
scoreWrong.textContent = '0';
```
With:
```javascript
scoreCorrect.textContent = '✓ 0';
scoreWrong.textContent = '✗ 0';
```

Add progress update in `showCard()` function (line ~2716), right after the existing `progress.textContent` line:
```javascript
scoreProgress.textContent = (index + 1) + ' / ' + shuffled.length;
```

Add progress reset in `applyFilter()` empty check (line ~2650), right after `progress.textContent = '0 / 0';`:
```javascript
scoreProgress.textContent = '0 / 0';
```

- [ ] **Step 5: Update mobile score-bar override in `@media (max-width: 520px)`**

Change existing `.score-bar { font-size: 12px; }` to:
```css
.score-bar { font-size: 11px; padding: 6px 12px; }
.card-front-content { padding: 0 16px; }
```

- [ ] **Step 6: Test in browser**

1. Score bar appears as a strip at top of flashcard
2. Shows correct/wrong counts and progress (e.g. "47 / 536")
3. Card content (word, POS, pronunciation) is still centered below the strip
4. Card flip works correctly — score bar on front only
5. Test all 5 themes — score bar background matches theme

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: integrate score bar as top strip inside flashcard"
```

---

## Chunk 3: Final Verification

### Task 6: Cross-Theme & Cross-Page Testing

- [ ] **Step 1: Test all 5 themes**

Switch through: default, cool, dark, midnight, forest. Verify:
- Island background, border, shadow match theme
- Active tab color matches theme
- Sub-filter buttons match their existing theme styling
- Score bar strip uses theme-appropriate background

- [ ] **Step 2: Test all 3 pages**

- Kelimeler: island toggle, all filter buttons work, quiz/match/grammar overlays launch
- Konular: unit switching works, topic cards expand
- Notlar: note filtering works, add note works

- [ ] **Step 3: Test responsive behavior**

- Desktop (>768px): island centered, fluid width
- Tablet (768px): verify layout
- Mobile (520px): island full width, touch targets >= 44px

- [ ] **Step 4: Final commit if any fixes needed**

```bash
git add index.html
git commit -m "fix: polish island nav and score bar across themes"
```
