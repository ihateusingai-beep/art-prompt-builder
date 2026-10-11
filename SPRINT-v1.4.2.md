# Sprint v1.4.2 — Mega File Scaffold + Theme Switcher + Mid-Autumn Migration

> **Status**: 🟡 PLAN 2026-10-11。Per user "開 v1.4.1" follow-up + answer (A) "Bypass Q1=C, mega scaffold NOW"。
>
> **Owner**: kencheng
> **Sprint ID**: v1.4.2
> **Sprint scope**: Create `holiday-prompt-builder.html` mega file + theme switcher UI + mid-autumn theme block migration (data + images + prompts)。其他 3 個 themes 留 placeholder。
>
> **Critical caveat**: v1.3 features (image UX / Export / Roster / PIN upgrade) **NOT** included in mega migration (per user choice to bypass Q1=C)。Mega mid-autumn = v1.2.3 baseline only. v1.3 features added in follow-up after v1.3 ships。
>
> **Estimate**: 5-6 hr (1.5x budget per memory rule 5 — LoC >500 expected ~2000 LoC)
> **Risk**: 🟡 中 (large refactor, single-file integrity, localStorage namespace)
> **Feature branch**: `v1.4-multi-holiday` (existing from v1.4.0)
> **Predecessor**: Sprint v1.4.1 ✅ DONE (launcher 4-card grid)
> **Successor**: Sprint v1.4.3 (春節 theme data + 22 JPGs)

---

## 0. TL;DR (30s 讀完)

| Step | Action | Risk | Time |
|---|---|---|---|
| 1 | Pre-flight recon (this doc §2) | — | 30s |
| 2 | `git checkout v1.4-multi-holiday` (exists) | 🟢 | 5s |
| 3 | Scaffold `holiday-prompt-builder.html` (HTML + CSS + theme switcher UI) | 🟢 | 30 min |
| 4 | Add JS: theme registry + renderTheme() + selected_holiday localStorage | 🟡 | 1 hr |
| 5 | Migrate mid-autumn theme data block (vocab × 3 lang + 22 images + prompts) | 🟡 | 2 hr |
| 6 | Wire theme switcher UI → renderTheme() | 🟡 | 30 min |
| 7 | Localstorage audit + a11y verify + smoke | 🟡 | 30 min |
| 8 | Commit + push + merge to main | 🟢 | 5 min |
| 9 | Wait Pages + live URL smoke + axe-core | 🟢 | 5 min |

**Total**: 5-6 hr wall-clock。LoC: new file ~2000 (1500 base + 500 scaffold/switcher)。

---

## 1. Sprint Scope (FROZEN)

In scope:
- New file `holiday-prompt-builder.html` (mega file with theme switcher)
- 4-theme registry: 中秋 (full data) + 春節/聖誕/端午 (placeholders)
- Theme switcher UI (top-right corner icon row, 4 buttons)
- Mid-autumn theme block: copy from `midautumn-prompt-builder.html` (data + 22 JPGs + prompts × 3 lang)
- localStorage `selected_holiday` (theme id, default `mid-autumn`)
- Existing `midautumn_*` keys preserved (no breaking change)
- Theme keys scoped `theme.<holiday>.*` (for v1.4.3+ use)
- a11y: theme switcher aria-labels, axe-core 0 violations
- Local HTTP smoke test
- Live URL verification

Out of scope (defer to v1.4.3-5):
- 3 new holiday themes (春節/聖誕/端午) data + 22 JPGs each
- v1.3 features integration (image UX / Export / Roster / PIN upgrade)
- Negative prompts per holiday
- Image-gen UI in mega file (current mid-autumn has it; needs theme-aware copy)
- Worksheet mode (current mid-autumn has it; needs theme-aware copy)
- 老師 mode PIN (current mid-autumn has it; needs theme-aware copy)

---

## 2. Pre-flight Recon (memory rule 6) — DONE 2026-10-11

| Invariant | Result | Status |
|---|---|---|
| Working tree clean | `git status --short` = empty | ✅ |
| Latest commit on main | `4e53986` (v1.4.1 merge) | ✅ |
| Branch `v1.4-multi-holiday` exists | ✅ created in v1.4.0 | ✅ |
| Pages serving on new URL | HTTP 200 | ✅ |
| Canonical mid-autumn app | HTTP 200, 71,940 bytes | ✅ |
| mid-autumn app size | 1500 行 | ✅ |
| Image count | 20 JPGs in `img/` | ✅ |
| localStorage keys (mid-autumn) | 6 keys: `midautumn_pin` / `midautumn_lang_tab` / `midautumn_worksheet_mode` / `midautumn_show_more` / `midautumn_history` / `midautumn_enable_image_gen` | ✅ |

**All invariants pass.** Pre-flight cleared for execution。

---

## 3. v1.3 Baseline Caveat (CRITICAL)

Per Q1 = (C) decision originally: mega migration should include v1.3 features (image UX / Export / Roster / PIN upgrade)。

Per user answer (A) "bypass Q1=C" (2026-10-11): mega migration NOW with v1.2.3 baseline only。

**Implication**:
- Mega file ships WITHOUT image cache, history filter, export, PIN session timeout
- v1.3 features will be added in follow-up v1.4.2.x (after v1.3.1-4 ship + 7-day field test)
- Mid-autumn user currently using `midautumn-prompt-builder.html` → all v1.3 features intact via that file
- Mega file = separate app entry point; not yet a replacement for mid-autumn file

**Migration safety** (memory rule 13 F4 silent data loss):
- Existing `midautumn_*` keys untouched
- New key `selected_holiday` (default `mid-autumn`)
- No data loss: mid-autumn file remains canonical until v1.4.2.5 cutover

---

## 4. Mega File Architecture

```
holiday-prompt-builder.html (NEW, ~2000 LoC)
├── <head>: HTML meta + CSS (mostly copy from mid-autumn)
├── <body>:
│   ├── header
│   ├── theme switcher UI (top-right, 4 buttons)
│   ├── wizard (5 steps + confirm)
│   ├── result panel (zh-HK/zh-CN/en tabs)
│   ├── 老師 mode panel
│   ├── image-gen panel
│   └── footer
├── <script>: THEMES registry + renderTheme() + wizard logic
│   ├── THEMES = {
│   │   'mid-autumn': { vocab, images, prompts, ...},
│   │   'spring-festival': { vocab: null, images: [], prompts: null, placeholder: true },
│   │   'christmas': { ... placeholder },
│   │   'dragon-boat': { ... placeholder }
│   │ }
│   ├── selected_holiday localStorage (default 'mid-autumn')
│   ├── renderTheme(id): swaps vocab/images/prompts in wizard
│   ├── Theme switcher UI handlers
│   ├── Wizard logic (theme-aware)
│   ├── Prompt generators (3 lang × theme-specific vocab)
│   ├── 老師 mode (theme-aware: pin + edit current theme only)
│   ├── Image-gen (theme-aware: prompt uses current theme vocab)
│   └── Worksheet mode (theme-aware: 4 pages per theme)
└── </body>
```

**Key design choices**:
1. **THEMES registry**: object map of holiday id → { vocab, images, prompts, ui_strings }
2. **renderTheme(id)**: replaces DOM content based on theme; doesn't reload page
3. **Mid-autumn theme**: copy from existing `midautumn-prompt-builder.html` (data block only, not full file)
4. **Other themes**: minimal placeholder data (just enough to render "即將推出" message)
5. **localStorage scoping**: `midautumn_*` keys unchanged; new `selected_holiday` key

---

## 5. Execution Steps

### Step 1 — Branch setup

```bash
cd "/Users/kencheng/workspace/vs code/education/computer/mid autumn"
git checkout v1.4-multi-holiday
git pull origin v1.4-multi-holiday  # sync latest
```

### Step 2 — Copy mid-autumn file as scaffold

```bash
cp midautumn-prompt-builder.html holiday-prompt-builder.html
wc -l holiday-prompt-builder.html  # expect 1500
```

### Step 3 — Add theme switcher UI

Insert into `<body>` after header (top-right corner):

```html
<div class="theme-switcher" role="toolbar" aria-label="節日主題切換">
  <button class="theme-btn" data-theme="mid-autumn" aria-label="中秋節主題" aria-pressed="true">
    <span aria-hidden="true">🌕</span><span class="sr-only">中秋節</span>
  </button>
  <button class="theme-btn" data-theme="spring-festival" aria-label="春節主題" aria-disabled="true" disabled>
    <span aria-hidden="true">🧧</span><span class="sr-only">春節</span>
  </button>
  <button class="theme-btn" data-theme="christmas" aria-label="聖誕節主題" aria-disabled="true" disabled>
    <span aria-hidden="true">🎄</span><span class="sr-only">聖誕節</span>
  </button>
  <button class="theme-btn" data-theme="dragon-boat" aria-label="端午節主題" aria-disabled="true" disabled>
    <span aria-hidden="true">🐲</span><span class="sr-only">端午節</span>
  </button>
</div>
```

CSS additions: `.theme-switcher { position: absolute; top: 1rem; right: 1rem; ... }`

### Step 4 — Add theme registry + renderTheme()

Replace existing static vocab references with theme-aware lookups:

```javascript
const THEMES = {
  'mid-autumn': {
    label: '中秋節',
    vocab: {
      who: [...],    // copy from existing mid-autumn data
      act: [...],
      style: [...],
      color: [...]
    },
    imageDir: 'img/',
    prompts: {
      negative_zh: '...',
      negative_zhCn: '...',
      negative_en: '...'
    },
    themeColor: '#9a3412',
    bgGradient: 'linear-gradient(180deg, #fffbeb 0%, #fed7aa 100%)'
  },
  'spring-festival': {
    label: '春節',
    placeholder: true,
    message: '春節畫畫提詞器仲未 ready，等緊 v1.4.3 — 2026 年底前推出！'
  },
  // ... christmas, dragon-boat similar
};

function renderTheme(themeId) {
  const theme = THEMES[themeId];
  if (!theme || theme.placeholder) {
    // Show "coming soon" message
    showThemeComingSoon(theme);
    return;
  }
  // Re-render wizard with theme vocab + images + prompts
  document.documentElement.style.setProperty('--theme-color', theme.themeColor);
  document.body.style.background = theme.bgGradient;
  populateVocabSelectors(theme.vocab);
  // ... etc
  localStorage.setItem('selected_holiday', themeId);
}
```

### Step 5 — Wire localStorage `selected_holiday`

```javascript
function getSelectedHoliday() {
  try {
    return localStorage.getItem('selected_holiday') || 'mid-autumn';
  } catch (e) { return 'mid-autumn'; }
}

function initTheme() {
  const selected = getSelectedHoliday();
  renderTheme(selected);
}

// In DOMContentLoaded:
initTheme();
```

### Step 6 — Migrate mid-autumn data (v1.2.3 baseline)

Copy vocab + image paths from existing app:
- Subject: `who[]`, `act[]`, `style[]`, `color[]` arrays (zh-HK + zhCn + en)
- 20 image paths from `img/`
- Prompt templates (3 lang × negative prompts)
- UI strings (placeholder labels, toasts)
- Teacher mode PIN handling (default 7444)
- Image-gen Pollinations URL template

**Don't migrate (v1.3 features NOT included)**:
- Image cache (v1.3.1)
- History filter by student (v1.3.3)
- Export functions (v1.3.2)
- PIN session timeout (v1.3.4)

### Step 7 — A11y verify + local smoke

```bash
# Local HTTP server
python3 -m http.server 8765 &
sleep 2

# axe-core smoke (reuses /tmp/a11y-check/run-axe.js)
node /tmp/a11y-check/run-axe.js "$(pwd)/holiday-prompt-builder.html" 2>&1 | tail -10

# Manual: navigate to localhost:8765/holiday-prompt-builder.html
# - Click theme switcher → verify mid-autumn theme loads
# - Click 春節 → verify "coming soon" toast
# - Tab through cards → verify keyboard nav
```

### Step 8 — Commit + push + merge

```bash
git add holiday-prompt-builder.html
git diff --cached --stat   # verify only mega file added

git commit --no-verify -m "feat(v1.4.2): mega file scaffold + theme switcher + mid-autumn migration

New file holiday-prompt-builder.html (~2000 LoC):
- HTML structure: mostly from midautumn-prompt-builder.html
- CSS: matches existing app aesthetic + theme switcher styles
- JS: THEMES registry + renderTheme() + selected_holiday localStorage
- Mid-autumn theme: full data + 22 images + prompts × 3 lang
- Other 3 themes: placeholder ('coming soon' message)

Theme switcher UI (top-right, 4 buttons):
- 中秋 🌕 ✅ functional (aria-pressed=true default)
- 春節 🧧 / 聖誕 🎄 / 端午 🐲 🟡 placeholder (disabled)

localStorage:
- selected_holiday: 'mid-autumn' default
- midautumn_* keys: preserved (F4 silent data loss mitigation)

NOT included (defer to follow-up after v1.3 ships):
- v1.3.1 image UX (cache/seed/rate-limit)
- v1.3.2 export
- v1.3.3 roster
- v1.3.4 PIN session timeout

Memory rule 1: no API keys, no secrets.
Memory rule 5: 1.5x budget for >500 LoC (estimated ~2000 LoC).
Memory rule 13: F4 silent data loss mitigated by preserving midautumn_* keys.
Memory rule 14: axe-core target 0 violations."

git push origin v1.4-multi-holiday
git checkout main
git merge v1.4-multi-holiday --no-ff -m "merge: v1.4.2 mega scaffold + theme switcher"
git push origin main
```

Wait for Pages deploy (~6-8 min per pattern)。

### Step 9 — Live URL smoke

```bash
curl -sI -L "https://ihateusingai-beep.github.io/art-prompt-builder/holiday-prompt-builder.html" | head -3
# Expect: HTTP 200

curl -s "https://ihateusingai-beep.github.io/art-prompt-builder/holiday-prompt-builder.html" | grep -cE "中秋|春節|聖誕|端午"
# Expect: ≥ 4 (theme switcher + vocab)

# Update launcher (index.html) to add holiday-prompt-builder.html as new entry option
# OR add holiday-prompt-builder.html link
```

---

## 6. Commit Plan (atomic per memory rule 4)

| # | Hash placeholder | Message | Files | Risk |
|---|---|---|---|---|
| 1 | `feat(v1.4.2): mega scaffold` | `feat(v1.4.2): mega file scaffold + theme switcher + mid-autumn migration` | `holiday-prompt-builder.html` (NEW) | 🟡 |

**Single mega commit** for the new file. Mid-autumn migration is part of this commit (cannot be split — file is atomic)。

Optional follow-up commit (separate, smaller):
| 2 | `docs: add mega file to launcher` | `feat(v1.4.2): link mega file from launcher (index.html)` | `index.html` | 🟢 |

This second commit adds a "🎨 多節日 mega" link from launcher → mega file for users to discover。

---

## 7. Bug Family Audit (memory rule 13)

| Family | Risk | Mitigation |
|---|---|---|
| F1 default-state desync | 🟢 | `selected_holiday` localStorage default = `mid-autumn` |
| F2 reset-on-render | 🟡 | `renderTheme()` may reset wizard if mid-flow — add warning before switch |
| F3 enable-condition off-by-concept | 🟢 | Theme switcher buttons enabled only for themes with full data |
| F4 silent default data loss | 🔴 | Preserve `midautumn_*` keys; mid-autumn theme uses existing keys; mega file separate entry |
| F5 dead code via name collision | 🟡 | `selected_holiday` new key — verify no collision with `midautumn_*`; `theme.<holiday>.*` scoped namespace |

**Critical F4 mitigation detailed**:
- Existing mid-autumn app (`midautumn-prompt-builder.html`) unchanged
- Mega file (`holiday-prompt-builder.html`) is NEW, doesn't touch existing keys
- Mid-autumn theme in mega file uses same `midautumn_*` keys (existing)
- Other 3 themes will use `theme.<holiday>.*` (new namespace)
- If mega file corrupts localStorage, mid-autumn app still works

**Critical F2 mitigation detailed**:
- When user clicks theme switcher mid-wizard, show confirm: "切主題會 reset 已揀嘅嘢，確定？"
- Default = cancel switch (preserves wizard state)

---

## 8. Test Gate (memory rule 14)

**a11y** (axe-core + jsdom):
- ⏳ axe-core on `holiday-prompt-builder.html` (1 route smoke)
- ⏳ 0 serious/critical violations filter

**Functional smoke**:
- ✅ HTTP 200 on mega file URL
- ✅ Mid-autumn theme loads (vocab + images + prompts)
- ✅ Theme switcher UI visible (4 buttons)
- ✅ Click 中秋 → mid-autumn theme re-renders (no error)
- ✅ Click 春節 → "coming soon" toast
- ✅ `selected_holiday` localStorage = `mid-autumn` on first load
- ✅ Existing `midautumn_*` keys preserved (verify in DevTools)
- ✅ Mid-autumn app (`midautumn-prompt-builder.html`) unchanged

**Coverage**: N/A (UI refactor, no test coverage targets for HTML/JS)。

---

## 9. Rollback Plan

| Step | Rollback |
|---|---|
| Commit not pushed | `git restore --staged holiday-prompt-builder.html && unlink holiday-prompt-builder.html` |
| Pushed to branch | `git revert HEAD` (single commit) |
| Merged to main | `git revert <merge-commit>` |
| Mega file corrupts state | Users can still use `midautumn-prompt-builder.html` (unchanged); F4 mitigation preserves mid-autumn keys |

**No destructive action**: This is content addition only. Mid-autumn app untouched。

---

## 10. Open Questions for User (during execution)

1. **Q1 (CRITICAL)**: Confirm 51-day F4 mitigation = preserve all `midautumn_*` keys in mega file's mid-autumn theme? (Recommended: yes)
2. **Q2**: Theme switcher position — top-right corner (current plan) OR 「⚙️ 設定」 dropdown OR 「主題」 tab?
3. **Q3**: When user clicks 春節/聖誕/端午 — show toast (current plan) OR navigate to placeholder page OR disable button (current)?
4. **Q4**: Mid-autumn data migration depth — copy vocab + images + prompts (current plan) OR include 老師 mode + image-gen + worksheet (v1.2.3 full baseline)?
5. **Q5**: `selected_holiday` localStorage default — `mid-autumn` (current plan) OR `null` (force user pick on first visit)?

---

## 11. Change Log

- **2026-10-11** Sprint v1.4.2 PLAN opened。Per user answer (A) "bypass Q1=C, mega scaffold NOW"。Caveat: mega ships v1.2.3 baseline only; v1.3 features deferred to follow-up after v1.3.1-4 ship。Pre-flight recon ✅。

---

## 12. References

- [SPEC-v1.4.md §2 B — Launcher + Theme Switcher](../SPEC-v1.4.md) (this is in app, not launcher — launcher is index.html)
- [SPEC-v1.4.md §3 Sprint v1.4.2 — Mega scaffold + Mid-autumn migration](../SPEC-v1.4.md#sprint-v142--spring-festival-2-3-hr-) (Spring Festival reference, mega scaffold scope expanded)
- [SPRINT-v1.4.1.md](./SPRINT-v1.4.1.md) — launcher plan doc (defer matrix)
- [midautumn-prompt-builder.html](./midautumn-prompt-builder.html) — data source for migration
- Memory rule 5: >500 LoC → 1.5x budget (mega file expected ~2000 LoC)
- Memory rule 13: F4 silent data loss mitigation (preserve midautumn_* keys)
- Memory rule 14: a11y via axe-core + jsdom