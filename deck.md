# /deck — Build a Single-Page HTML Presentation

Build a polished, single-file HTML carousel presentation following the exact
viewport, layout, typography, interaction, and normalization standards derived
from the Wyndham MCP and Red Roof Inn demo decks.

## Usage

```
/deck <brief description of what to build>
```

Examples:
- `/deck MuleSoft API governance demo for Capital One — 5 slides`
- `/deck Agentforce ROI pitch for healthcare customer — dark theme`
- `/deck product roadmap overview — light theme, 4 slides`

---

## Standards to apply — always follow these exactly

### 1. Viewport & Carousel Framework

The entire page is a fixed-position carousel. Slides never scroll the page —
interactive content scrolls *inside* its own container.

```css
body { overflow: hidden; }

#carousel {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  overflow: hidden;
}

.slide {
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  display: flex;
  flex-direction: column;
  overflow: hidden;                          /* NEVER overflow-y: auto here */
  transform: translateX(100%);
  transition: transform 0.45s cubic-bezier(0.4, 0, 0.2, 1);
  will-change: transform;
  visibility: hidden;
  pointer-events: none;
}
.slide.active { transform: translateX(0);    visibility: visible; pointer-events: auto; }
.slide.prev   { transform: translateX(-100%); }
```

### 2. Slide Inner — Content Centering

Sparse content slides (title, summary, closing) center their content vertically.
Dense or interactive slides (chat, tables, multi-column) anchor to top.

```css
.slide-inner {
  max-width: 1100px;
  width: 100%;
  margin: 0 auto;
  padding: 70px 48px 40px;   /* top pad clears 50px nav bar */
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;   /* default: centered */
}

/* For any slide with a lot of content or interactive panels: */
.slide-dense .slide-inner,
.slide-chat  .slide-inner,
.slide-scene .slide-inner {
  justify-content: flex-start;
  overflow: hidden;
  height: 100%;
}
```

### 3. Interactive Panel Height — The Flex Chain Rule

Chat windows, data grids, and scrollable panels must fill available height
without overflowing the viewport. Every ancestor in the chain needs
`flex:1; min-height:0`. The scrollable leaf node gets `overflow-y:auto`.

```css
/* Two-column interactive layout */
.panel-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 24px;
  align-items: stretch;
  flex: 1;             /* fills slide-inner */
  min-height: 0;       /* prevents flex children ignoring overflow */
  overflow: hidden;
  margin-top: 10px;
}

/* Left column: passes height to chat window */
.panel-layout > div:first-child {
  display: flex;
  flex-direction: column;
  min-height: 0;
}

/* Chat / data window */
.chat-window {
  flex: 1;
  min-height: 0;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  border-radius: 14px;
}

/* The scrollable message body inside the window */
.chat-body {
  flex: 1;
  min-height: 0;
  overflow-y: auto;
  scroll-behavior: smooth;
  padding: 12px 14px;
}

/* Right value/info panel */
.value-panel {
  overflow-y: auto;
  min-height: 0;
}
```

**Rule:** Never use a fixed `height: Npx` on any chat body or data panel.
Always use `flex:1; min-height:0; overflow-y:auto`.

### 4. Navigation Bar

Fixed, 50px, always on top. Contains: brand name, separator, prev/next arrows,
dot indicators, slide label, counter, keyboard hint.

```html
<nav class="nav-bar" id="nav-bar">
  <div class="nav-brand">Brand <span>× Product</span></div>
  <div class="nav-sep"></div>
  <button class="nav-arrow" id="btn-prev">&#8592;</button>
  <div class="nav-dots" id="nav-dots"></div>
  <button class="nav-arrow" id="btn-next">&#8594;</button>
  <div class="nav-sep"></div>
  <div class="nav-label" id="nav-label"></div>
  <div class="nav-counter" id="nav-counter"></div>
  <div class="nav-sep"></div>
  <div class="nav-hint"><kbd>←</kbd><kbd>→</kbd> to navigate</div>
</nav>
```

```css
.nav-bar {
  position: fixed;
  top: 0; left: 0; right: 0;
  z-index: 200;
  height: 50px;
  background: rgba(BRAND_COLOR, 0.97);
  backdrop-filter: blur(8px);
  border-bottom: 1px solid rgba(ACCENT, 0.3);
  display: flex; align-items: center;
  padding: 0 20px; gap: 12px;
}
```

### 5. Carousel JavaScript Engine

Always use this exact engine. Do not invent alternatives.

```js
const SLIDES = [
  { id: 's1', label: 'Slide Label' },
  // ... one entry per slide
];

let current = 0;

function buildNav() {
  const dots = document.getElementById('nav-dots');
  SLIDES.forEach((s, i) => {
    const d = document.createElement('div');
    d.className = 'nav-dot' + (i === 0 ? ' active' : '');
    d.title = s.label;
    d.onclick = () => goTo(i);
    dots.appendChild(d);
  });
}

function updateNav() {
  document.querySelectorAll('.nav-dot').forEach((d, i) =>
    d.classList.toggle('active', i === current));
  document.getElementById('nav-label').textContent   = SLIDES[current].label;
  document.getElementById('nav-counter').textContent = `${current + 1} / ${SLIDES.length}`;
  document.getElementById('btn-prev').disabled = current === 0;
  document.getElementById('btn-next').disabled = current === SLIDES.length - 1;
}

function goTo(idx) {
  if (idx < 0 || idx >= SLIDES.length || idx === current) return;
  const prev = current;
  current = idx;
  SLIDES.forEach((s, i) => {
    const el = document.getElementById(s.id);
    el.classList.remove('active', 'prev');
    if (i === current) el.classList.add('active');
    else if (i === prev) el.classList.add('prev');
  });
  updateNav();
  // Optional: trigger per-slide init (e.g. chat lazy init)
  // if (idx === 2) initChatSlide();
}

document.getElementById('btn-prev').onclick = () => goTo(current - 1);
document.getElementById('btn-next').onclick = () => goTo(current + 1);
document.addEventListener('keydown', e => {
  if (e.key === 'ArrowRight') goTo(current + 1);
  if (e.key === 'ArrowLeft')  goTo(current - 1);
});

// NOTE: do NOT call goTo(0) here — the idx===current guard makes it a no-op
// when current starts at 0. Instead, initialize the first slide explicitly:
buildNav();
document.getElementById(SLIDES[0].id).classList.add('active');
updateNav();
```

### 6. Typography Scale

Use `clamp()` for headings so they scale across viewport widths.
Never use fixed `px` for display text.

```css
/* Eyebrow / label (above a heading) */
.eyebrow {
  font-size: 11px; font-weight: 700;
  text-transform: uppercase; letter-spacing: 2px;
  color: ACCENT_COLOR; margin-bottom: 10px;
}

/* Title slide H1 */
.title-h1 {
  font-size: clamp(36px, 5vw, 68px);
  font-weight: 900; line-height: 1.05;
  letter-spacing: -2px; margin-bottom: 12px;
}

/* Section H1 (inside slides) */
.slide-h1 {
  font-size: clamp(26px, 3.2vw, 42px);
  font-weight: 800; line-height: 1.15;
  letter-spacing: -0.5px; margin-bottom: 8px;
}

/* Descriptor / subtitle under H1 */
.slide-desc {
  font-size: 14px; color: MUTED_COLOR;
  line-height: 1.65; max-width: 680px;
  margin-bottom: 0;
}

/* Card body copy */
.card-body { font-size: 12px; line-height: 1.55; color: MUTED_COLOR; }

/* Data label (metrics, field names) */
.data-label {
  font-size: 9.5px; font-weight: 700;
  text-transform: uppercase; letter-spacing: 0.8px;
  color: MUTED_COLOR;
}
```

### 7. Color Token System

Define all colors as CSS custom properties on `:root`. Never hardcode brand
colors inline — always use `var(--token)`.

Minimum token set for any deck:
```css
:root {
  --brand:        #XXXXXX;   /* primary brand color */
  --brand-light:  #XXXXXX;   /* lighter tint for text/accents */
  --brand-dark:   #XXXXXX;   /* darker shade for hover/depth */
  --accent:       #XXXXXX;   /* secondary accent (gold, blue, etc) */

  /* Dark surface system (use for dark decks) */
  --bg:           #0d1117;
  --surf:         #161b22;
  --surf2:        #21262d;
  --border:       #30363d;
  --text:         #e6edf3;
  --muted:        #8b949e;

  /* Light surface system (use for light slides / chat panels) */
  --light-bg:     #f0f4f9;
  --light-surf:   #ffffff;
  --light-border: #e4e8ef;
  --light-text:   #1a2a3a;
  --light-muted:  #556070;
}
```

### 8. Cards — Self-Contained UI Blocks

Cards must always be visually complete. The response/body area in a card
must fill the card's full rounded rectangle — use `display:flex;
flex-direction:column` on the card, and `flex:1` on the content area.

```css
.card {
  border-radius: 10px;
  overflow: hidden;           /* clips children to rounded corners */
  border: 1px solid var(--border);
  display: flex;
  flex-direction: column;     /* required so children can fill height */
}

.card-header {
  background: var(--surf2);
  border-bottom: 1px solid var(--border);
  padding: 9px 12px;
  flex-shrink: 0;             /* header never shrinks */
}

.card-body {
  background: var(--surf);
  padding: 9px 12px;
  flex: 1;                    /* fills remaining card height */
}
```

### 9. Interactive Chat Pattern

When a slide has an interactive chat demo, lazy-initialize it on first visit
(not on page load) to avoid JS running for slides the user never opens.

```js
let chatInitialized = false;

function initChat() {
  if (chatInitialized) return;
  chatInitialized = true;

  const msgs  = document.getElementById('chat-messages');
  const input = document.getElementById('chat-input');
  const send  = document.getElementById('chat-send');
  let locked  = false;

  function scroll() { msgs.scrollTop = msgs.scrollHeight; }

  function addUser(text) {
    const d = document.createElement('div');
    d.className = 'msg-user';
    d.innerHTML = `<div class="bubble-user">${text}</div>`;
    msgs.appendChild(d); scroll();
  }

  function addBot(html, delay = 1000) {
    return new Promise(res => {
      const t = document.createElement('div');
      t.className = 'msg-bot';
      t.innerHTML = `<div class="bot-avatar">✦</div>
        <div class="bubble-bot">
          <div class="typing-dots">
            <div class="dot"></div><div class="dot"></div><div class="dot"></div>
          </div>
        </div>`;
      msgs.appendChild(t); scroll();
      setTimeout(() => {
        t.querySelector('.bubble-bot').innerHTML = html;
        scroll(); res();
      }, delay);
    });
  }

  function addToolCall(name, result) {
    return new Promise(res => setTimeout(() => {
      const d = document.createElement('div');
      d.className = 'tool-call';
      d.innerHTML = `🔗 <strong>${name}</strong> → <span>${result}</span>`;
      msgs.appendChild(d); scroll(); res();
    }, 400));
  }

  function setChips(options) {
    const c = document.getElementById('chat-chips');
    c.innerHTML = '';
    c.style.display = options.length ? 'flex' : 'none';
    options.forEach(opt => {
      const b = document.createElement('div');
      b.className = 'chip';
      b.textContent = opt;
      b.onclick = () => handleInput(opt);
      c.appendChild(b);
    });
  }

  async function handleInput(text) {
    if (locked) return;
    setChips([]); addUser(text); locked = true;
    // ── state machine: add cases here ──
    // await addBot('response', 1000);
    // await addToolCall('tool_name', 'result');
    // setChips(['Follow-up option']);
    locked = false;
  }

  send.onclick = () => {
    const v = input.value.trim();
    if (v) { input.value = ''; handleInput(v); }
  };
  input.addEventListener('keydown', e => {
    if (e.key === 'Enter') { const v = input.value.trim(); if (v) { input.value = ''; handleInput(v); } }
  });

  // Start with greeting
  addBot('Hello! How can I help?', 600).then(() =>
    setChips(['Option 1', 'Option 2']));
}
```

Typing animation CSS:
```css
.typing-dots { display: flex; gap: 4px; padding: 4px 0; }
.dot {
  width: 6px; height: 6px; border-radius: 50%;
  background: var(--muted); animation: bounce 1.2s infinite;
}
.dot:nth-child(2) { animation-delay: 0.2s; }
.dot:nth-child(3) { animation-delay: 0.4s; }
@keyframes bounce {
  0%, 80%, 100% { transform: translateY(0); opacity: 0.4; }
  40%           { transform: translateY(-5px); opacity: 1; }
}
```

### 10. Tab Panels (multi-scenario slides)

```js
function switchTab(tab) {
  const tabs   = ['tab1', 'tab2', 'tab3'];      // tab IDs
  const panels = ['panel1', 'panel2', 'panel3']; // panel IDs
  tabs.forEach(t => document.getElementById(t).className = 'tab-btn');
  panels.forEach(p => document.getElementById(p).style.display = 'none');
  document.getElementById('tab-' + tab).className = 'tab-btn active';
  document.getElementById('panel-' + tab).style.display = 'grid'; // or 'flex'
}
```

Tab button active states: use a branded class (`active`, `active-google`,
`active-claude`, etc.) so each tab can have its own highlight color.

### 11. Slide Type Checklist

For every slide, confirm:
- [ ] `overflow: hidden` on `.slide` (never `overflow-y: auto`)
- [ ] Sparse slides: `justify-content: center` on `.slide-inner`
- [ ] Dense/interactive slides: `justify-content: flex-start` + `overflow: hidden` + `height: 100%` on `.slide-inner`
- [ ] All chat/data panels: `flex:1; min-height:0; overflow-y:auto` — no fixed heights
- [ ] All cards: `display:flex; flex-direction:column` + `overflow:hidden` on container, `flex:1` on body
- [ ] Typography uses `clamp()` for headings
- [ ] All colors reference `var(--token)` — no hardcoded hex in component CSS
- [ ] Nav bar is 50px fixed, `z-index:200`, top padding on `slide-inner` is ≥70px
- [ ] Carousel JS uses exact `goTo()` engine with `.active` / `.prev` class toggling
- [ ] Keyboard navigation: `ArrowLeft` / `ArrowRight`
- [ ] Chat state machines lazy-initialize on first slide visit
- [ ] Single `index.html` file — no external dependencies, no CDN fonts unless requested

### 12. Client-Side Password Lock

Paste this block as the **very first thing inside `<body>`** (before nav, carousel, everything).
It renders a full-screen gate, checks the password, and stores a session token in
`sessionStorage` so the user isn't re-prompted on refresh within the same tab.

Hash the password with SHA-256 so the plaintext never appears in source.
Generate the hash: `echo -n "yourpassword" | shasum -a 256`

```html
<!-- ── Password Gate ───────────────────────────────────────── -->
<div id="pw-gate" style="
  position:fixed;inset:0;z-index:9999;
  background:#0d1117;
  display:flex;align-items:center;justify-content:center;
  font-family:system-ui,sans-serif;">
  <div style="
    background:#161b22;border:1px solid #30363d;border-radius:14px;
    padding:40px 48px;text-align:center;max-width:360px;width:90%;">
    <div style="font-size:22px;font-weight:800;color:#e6edf3;margin-bottom:6px;">
      BRAND_NAME
    </div>
    <div style="font-size:13px;color:#8b949e;margin-bottom:28px;">
      Enter the access password to continue
    </div>
    <input id="pw-input" type="password" placeholder="Password"
      style="
        width:100%;box-sizing:border-box;
        background:#0d1117;border:1px solid #30363d;border-radius:8px;
        color:#e6edf3;font-size:14px;padding:10px 14px;outline:none;
        margin-bottom:12px;"
      onkeydown="if(event.key==='Enter')checkPw()"/>
    <button onclick="checkPw()" style="
      width:100%;background:BRAND_COLOR;border:none;border-radius:8px;
      color:#fff;font-size:14px;font-weight:700;padding:10px;cursor:pointer;">
      Continue →
    </button>
    <div id="pw-err" style="color:#e8435a;font-size:12px;margin-top:10px;min-height:16px;"></div>
  </div>
</div>

<script>
(function() {
  // SHA-256 hash of the password — generate with: echo -n "pass" | shasum -a 256
  const HASH = 'PASTE_SHA256_HASH_HERE';
  const KEY  = 'deck_auth';

  if (sessionStorage.getItem(KEY) === HASH) unlock();

  async function checkPw() {
    const val = document.getElementById('pw-input').value;
    const buf = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(val));
    const hex = Array.from(new Uint8Array(buf)).map(b => b.toString(16).padStart(2,'0')).join('');
    if (hex === HASH) { sessionStorage.setItem(KEY, HASH); unlock(); }
    else {
      const err = document.getElementById('pw-err');
      err.textContent = 'Incorrect password.';
      document.getElementById('pw-input').value = '';
      setTimeout(() => err.textContent = '', 2500);
    }
  }

  function unlock() {
    const gate = document.getElementById('pw-gate');
    if (gate) gate.remove();
  }

  window.checkPw = checkPw;
})();
</script>
<!-- ── End Password Gate ───────────────────────────────────── -->
```

**To use:**
1. Run `echo -n "yourpassword" | shasum -a 256` to get the hash
2. Paste hash into `HASH = '...'`
3. Replace `BRAND_NAME` and `BRAND_COLOR` to match the deck theme

For `salesforce1` the SHA-256 hash is:
`61c7c0702198c97bf59ce68c818d9ec9d3eccda9246948e2ef02abd7f7627c1b`

### 13. Responsive

Every deck must be mobile-readable. Decks are shared via URL — customers
will open them on phones. Use two breakpoints: tablet (≤900px) and mobile (≤600px).

```css
/* ── Tablet (≤900px) ─────────────────────────────── */
@media (max-width: 900px) {
  .slide-inner { padding: 60px 28px 32px; }

  /* Hide keyboard hint in nav */
  .nav-hint { display: none; }
  .nav-bar  { gap: 8px; padding: 0 14px; }

  /* Stack arch diagrams vertically */
  .arch-diagram, .panel-layout { grid-template-columns: 1fr !important; }
  .arch-arrow { transform: rotate(90deg); margin: 0 auto; }

  /* Chat: stack columns, hide sidebar */
  .chat-layout   { grid-template-columns: 1fr; }
  .chat-sidebar  { display: none; }

  /* Content grids: 2 cols on tablet */
  .caps-grid, .what-grid { grid-template-columns: 1fr 1fr; }
  .next-grid              { grid-template-columns: 1fr 1fr; }
  .obj-grid               { grid-template-columns: 1fr; }
}

/* ── Mobile (≤600px) ─────────────────────────────── */
@media (max-width: 600px) {

  /* Slides stack vertically; page scrolls. No translateX —
     avoids horizontal overflow clipping on small screens. */
  body      { overflow-x: hidden; overflow-y: auto; }
  #carousel { position: relative; height: auto; overflow: visible; }
  .slide    { position: relative; transform: none !important;
              transition: none; visibility: visible;
              pointer-events: auto; display: none; width: 100%; }
  .slide.active { display: flex; }
  .slide.prev   { display: none; }

  /* Let slide-inner flow naturally */
  .slide-chat .slide-inner,
  .slide-inner { overflow: visible; height: auto; flex: none; }

  /* Nav: hide dots entirely (counter is enough) + hide label */
  .nav-dots    { display: none; }
  .nav-label   { display: none; }
  .nav-brand   { font-size: 13px; }
  .nav-counter { font-size: 11px; }

  .slide-inner { padding: 58px 16px 24px; }

  /* Title slide: stack hero + video vertically */
  .title-hero-grid {
    display: flex !important;
    flex-direction: column !important;
    gap: 24px !important;
  }

  /* Arch compare: stack columns vertically */
  .arch-compare-grid {
    display: flex !important; flex-direction: column !important; gap: 8px !important;
  }

  /* Headlines scale down */
  .title-h1   { font-size: 28px !important; }
  .slide-h1   { font-size: 20px !important; }
  .slide-desc { font-size: 13px; }

  /* Tags wrap tightly */
  .title-tag  { font-size: 11px; padding: 4px 10px; }
  .logo-badge { font-size: 10px; padding: 4px 9px; }

  /* Chat tabs: horizontally scrollable, no wrap */
  .tab-row { overflow-x: auto; flex-wrap: nowrap; padding-bottom: 4px; }
  .tab-btn { font-size: 11px; padding: 5px 10px; white-space: nowrap; flex-shrink: 0; }

  /* Chat windows: fixed height so input bar is always visible */
  .chat-window  { flex: none; height: 400px; min-height: 0; display: flex; flex-direction: column; }
  .chat-body    { flex: 1; min-height: 0; overflow-y: auto; height: auto; }
  .chat-chips   { flex-shrink: 0; }
  .chat-input-bar { flex-shrink: 0; }

  /* All multi-col grids collapse to single col */
  .caps-grid, .what-grid, .next-grid,
  .obj-grid, .panel-layout,
  .assets-row-grid { grid-template-columns: 1fr !important; }

  /* Closing bar stacks */
  .closing-bar { flex-direction: column; text-align: center; gap: 6px; }
}
```

**Rules:**
- Never use a fixed `height` or `min-height` that would cause overflow on small screens
- **Never use `style="display:grid;grid-template-columns:..."` inline on elements that need to respond to breakpoints.** Always assign a named class (e.g. `title-hero-grid`, `arch-compare-grid`, `assets-row-grid`) and define the grid layout in CSS. Inline styles override media queries.
- Chat windows on mobile need `flex: none; height: 400px` with `flex:1` body — if the container has no explicit height to flex against, the input bar will be pushed off-screen
- Always test the title slide at 390px width (iPhone 15 viewport)

### 14. Named Grid Classes — Required for Responsiveness

**Never use `style="display:grid;grid-template-columns:..."` on elements that
must collapse on mobile.** Inline styles have higher specificity than media
queries — the breakpoint override will be silently ignored.

Assign a named class for every multi-column layout that needs to respond:

```html
<!-- WRONG — media query can never override this -->
<div style="display:grid;grid-template-columns:1fr 1fr;gap:56px;">

<!-- RIGHT — class can be overridden in @media block -->
<div class="title-hero-grid">
```

```css
/* Define the grid in CSS */
.title-hero-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 56px;
  align-items: center;
}

/* Collapse cleanly at mobile breakpoint */
@media (max-width: 600px) {
  .title-hero-grid {
    display: flex !important;
    flex-direction: column !important;
    gap: 24px !important;
  }
}
```

Common named grid classes to define in every deck:
- `.title-hero-grid` — title slide two-column (headline + media)
- `.arch-compare-grid` — arch comparison layout (left `fr` / center arrow / right `fr`)
- `.assets-row-grid` — 2-col asset/card row grids
- `.panel-layout` — interactive two-panel (chat left, value right) — already defined in section 3

Only leave inline grids for tiny decorative elements (e.g. a 2-badge pill row
inside a card) that will never need to collapse.

---

## What to deliver

1. A single `index.html` file at the path the user specifies (default: working directory)
2. All CSS inline in `<style>`, all JS inline in `<script>` — zero external deps
3. Ask the user for: slide count, brand colors, theme (dark/light/mixed), any
   interactive chat scenarios needed — if not provided, make reasonable choices
   and note them

User's request: $ARGUMENTS
