# /deck — Claude Code Skill for Customer-Facing HTML Decks

A [Claude Code slash command](https://docs.anthropic.com/en/docs/claude-code/slash-commands) that builds polished, single-file HTML carousel presentations — the kind you can deploy to GitHub Pages and share directly with customers.

## What it does

`/deck` generates a complete `index.html` file with:

- **Slide carousel** — keyboard navigation (←/→), dot indicators, slide counter, animated transitions
- **Client-side password gate** — SHA-256 hashed, no plaintext in source, session-persisted
- **Dark theme by default** — with a CSS token system for brand color customization
- **Interactive chat demos** — lazy-initialized Agentforce/AI agent simulations with typing animations, tool call display, and suggested chips
- **Architecture diagrams** — node/arrow layouts for showing data flows and integrations
- **Responsive layout** — two-breakpoint system (tablet ≤900px, mobile ≤600px) that actually works
- **Zero dependencies** — all CSS and JS inline, no CDN, no build step

## Usage

```
/deck <brief description of what to build>
```

**Examples:**
```
/deck MuleSoft API governance demo for Capital One — 5 slides
/deck Agentforce ROI pitch for healthcare customer — dark theme
/deck Red Roof Inn franchise ops demo — 8 slides with chat demo
```

The skill will ask for (or make reasonable defaults about): slide count, brand colors, theme, and any interactive chat scenarios needed.

## Output

A single `index.html` file that you can open locally or deploy to any static host. No build step, no dependencies, no server required.

To deploy to GitHub Pages:
1. Create a repo (e.g. `my-customer-deck`)
2. `git init && git add index.html && git commit -m "init deck"`
3. `git push -u origin main`
4. Enable Pages in repo Settings → Pages → Deploy from branch `main`

## Design standards

The skill enforces a consistent set of design patterns derived from production Salesforce customer decks:

| Standard | Rule |
|---|---|
| Viewport | Fixed-position carousel, slides never scroll the page |
| Content centering | Sparse slides centered, dense/interactive slides anchor to top |
| Interactive panels | Flex chain rule: `flex:1; min-height:0` on every ancestor, `overflow-y:auto` on leaf |
| Typography | `clamp()` for all display headings — never fixed `px` on H1/H2 |
| Color | CSS custom properties (`var(--token)`) — no hardcoded hex in component CSS |
| Cards | `display:flex; flex-direction:column; overflow:hidden` + `flex:1` on body |
| Grids | Named CSS classes for all responsive grids — never `style="display:grid..."` on elements that need to collapse |
| Nav bar | 50px fixed, `z-index:200`, top padding ≥70px on slide-inner |
| Mobile | Hide nav dots at ≤600px (counter only); slides use display:none/flex |
| Password | SHA-256 client-side gate in sessionStorage, no plaintext in source |

## Password

The default demo password is `salesforce1`.

SHA-256 hash: `61c7c0702198c97bf59ce68c818d9ec9d3eccda9246948e2ef02abd7f7627c1b`

To use a different password:
```bash
echo -n "yourpassword" | shasum -a 256
```
Paste the resulting hash into the `HASH` constant in the password gate block.

## Examples built with /deck

- **[Wyndham × Salesforce MCP — 5-Minute Recap](https://ploughran.github.io/wyndham-mcp-5min/)** — Post-SIC speed recap deck with Agentforce chat demos, architecture diagrams, and a live Vidyard embed
- **[Red Roof Inn Agentforce Demo](https://ploughran.github.io/red-roof-demo/)** — Franchise operations storyboard with interactive scenario walkthroughs

## Installation

This is a Claude Code slash command. To install:

1. Copy `deck.md` to `~/.claude/commands/deck.md`
2. In Claude Code, type `/deck <your description>`

Claude Code discovers all `.md` files in `~/.claude/commands/` automatically.
