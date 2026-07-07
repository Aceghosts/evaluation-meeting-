# TEMPLATE.md — "Ember Portfolio" replication guide

A complete spec for rebuilding this exact site as a personal portfolio for **anyone**.
Hand this file (plus a copy of `index.html` as reference) to Claude Code or a developer
and say: "Build this template with my content." Every mechanic, token, and magic number
is documented below. The "Content slots" section at the end lists everything you swap out.

---

## 1. Architecture

- **One self-contained `index.html`** (~190KB). All CSS in a single `<style>` block,
  all JS in a single `<script>` block (IIFE, `"use strict"`). No build step, no framework.
  The dev loop is: open the file in a browser.
- Portrait photo embedded as **base64 PNG** in an `<img id="portraitImg">` tag.
- Deployment: push to a GitHub repo → Vercel auto-deploys `main` as a static site.
  No config needed; Vercel serves `index.html` at the root.

### CDN dependencies (exact working URLs)

```html
<link href="https://fonts.googleapis.com/css2?family=Bricolage+Grotesque:opsz,wght@12..96,300..800&family=Instrument+Sans:wght@400;500;600&family=Space+Grotesk:wght@400;500;700&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lenis@1.1.14/dist/lenis.min.js"></script>
```

> ⚠️ Do NOT use `cdnjs.cloudflare.com/ajax/libs/lenis/...` — it 404s silently
> (the code guards with `typeof Lenis !== "undefined"`, so smooth scroll just
> never activates and you won't see an error). Use jsdelivr as above.

Add `<link rel="preconnect">` for `fonts.googleapis.com` and `fonts.gstatic.com` (crossorigin).

---

## 2. Design tokens

All colors are CSS custom properties on `:root`. To re-theme the whole site,
change ONLY these — every gradient, glow, and border derives from them.

| Token | Value | Role |
|---|---|---|
| `--bg` | `#0B0908` | page background (warm near-black) |
| `--surface` | `#161110` | cards, panels |
| `--surface-2` | `#1E1715` | raised panels |
| `--line` | `rgba(244,237,228,.09)` | hairline borders |
| `--cream` | `#F4EDE4` | primary text |
| `--muted` | `#9A8E85` | secondary text |
| `--ember` | `#FF5A1F` | primary accent |
| `--amber` | `#FFB35C` | gradient partner accent |
| `--deep` | `#7A2408` | dark accent for big gradients |

- Accent gradient is always `linear-gradient(96–150deg, var(--ember), var(--amber))`.
- Re-theming rule: keep `--bg` near-black and low-chroma, keep `--cream`/`--muted`
  warm neutrals tinted toward the accent hue, pick `--ember`/`--amber` as two
  adjacent hues (~25–35° apart), and `--deep` as a very dark version of `--ember`.

### Typography

| Font | Use | Details |
|---|---|---|
| Bricolage Grotesque 600–800 | display headings | letter-spacing −.02 to −.035em; italic spans inside headlines get the gradient text treatment |
| Instrument Sans 400–600 | body copy | 15–17px, line-height ~1.7, `--muted` color |
| Space Grotesk 400–700 | eyebrows / labels / numbers | 11–13px, letter-spacing .1–.22em, UPPERCASE |

### Motion language (non-negotiable)

- Never `linear` or default `ease` on UI motion.
- `--ease-out: cubic-bezier(.22,1,.36,1)` for reveals (GSAP: `power3.out` / `expo.out`).
- `--ease-soft: cubic-bezier(.4,0,.2,1)` for hovers/micro-interactions.
- Durations: hero entrance moves 0.8–1.4s, scroll reveals ~0.9s, hovers 0.2–0.3s,
  stagger between siblings 60–130ms.
- Animate ONLY `transform` and `opacity` in anything scroll- or loop-driven.
- Everything decorative is gated behind `prefers-reduced-motion` — a single
  `const reduce = matchMedia("(prefers-reduced-motion: reduce)").matches` flag in JS
  plus a CSS `@media` block that kills keyframe animations.
- Custom cursor + magnetic buttons only when `matchMedia("(pointer: fine)")` — never on touch.
- No FOUC: set all initial hidden states with `gsap.set(...)` before building the
  entrance timeline.

---

## 3. Global chrome

- **Custom cursor**: a 10px `border-radius:50%` accent dot, `position:fixed`,
  lerped toward the mouse each rAF; scales up over links/buttons. `(pointer: fine)` only.
- **Film grain**: a full-viewport pseudo-element with a tiny repeating noise pattern,
  animated by a `@keyframes grain` step jitter at very low opacity. Subtle — you should
  barely notice it.
- **Lenis smooth scroll**: `new Lenis({ duration: 1.15, smoothWheel: true })`, driven by
  its own rAF loop, wired to `ScrollTrigger.update` via `lenis.on("scroll", ...)`.
  Anchor links call `lenis.scrollTo(target, {offset:-70})` (fallback: `scrollIntoView`).
- **Magnetic buttons**: on pointermove within a small radius, translate the button a
  fraction of the cursor offset (~0.25×), spring back to 0 on leave. Fine pointers only.

---

## 4. Section-by-section spec (top to bottom)

### 4.1 Nav
Fixed top bar: logo (name + 9px gradient dot with glow) left, anchor links center,
pill CTA "Get in touch" right (gradient background, magnetic). After 40px of scroll,
the nav gains a blurred/darkened background and shrinks slightly (toggle a class).

### 4.2 Hero (100vh)
- **Headline**: 2–3 masked lines, e.g. "Designing brands that / *refuse* / to be boring."
  Each line is `.line > span`; the span translates up from 110% inside an
  `overflow:hidden` parent. **Give the span padding + negative margin insets** so
  italic glyph overhangs don't clip — keep this pattern for every masked reveal.
  The emotional word is an italic span with gradient text.
- **Atmosphere**: two large blurred gradient orbs (`border-radius:50%`,
  `filter:blur(90px)`, `will-change:transform`) + a thin ring
  (`min(46vw,560px)` circle, 1px `rgba(amber,.18)` border, with an inner dashed
  ring spinning `60s linear infinite`). All parallax elements carry `data-mx` /
  `data-my` attributes; one mousemove listener lerps them toward
  `(cursorX * mx, cursorY * my)`.
- **Portrait**: arch shape (border-radius: 50% 50% 0 0 on the top), ember→amber
  gradient as the arch background, B&W (or natural) cut-out photo of you on top,
  a bottom fade mask into the page bg, and a small glass badge
  ("● Available for collabs" — green 7px pulsing dot). Sits right side; on
  desktop ≥981px the text column is width-capped so it never runs under the portrait.
- **Skill tags**: 4 numbered chips along the hero bottom (#01 Branding, #02 …).
- **Entrance timeline**: nav fades, lines mask in stagger, portrait slides/fades,
  orbs breathe in — 0.8–1.4s, `expo.out`.

### 4.3 Brand marquee
A strip of brand/client names separated by accent dots. CSS:
`.marquee { display:flex; width:max-content; animation: scrollx 34s linear infinite }`
with `@keyframes scrollx { to { transform: translateX(-50%) } }` — JS duplicates the
content once so −50% loops seamlessly. Pause on hover
(`.marquee-sec:hover .marquee { animation-play-state: paused }`). Edge fade masks.
(This is the one acceptable `linear` — constant-speed conveyor.)

### 4.4 About
Two-column: bio copy left (an eyebrow label, a heading with one italic gradient word,
2–3 paragraphs), a stats + skills card right:
- **Count-up stats** (e.g. 4+ years / 15+ brands / 6 agencies): numbers tween from 0
  when scrolled into view (ScrollTrigger `once`).
- **Skill bars**: each bar's fill is `scaleX` 0→pct with stagger when in view.
- Card has a decorative 280px blurred gradient circle bleeding off its corner.

### 4.5 Work — the 3D fan carousel (the signature piece)
Filter pills (All / cat1 / cat2 / cat3) + a `#stage` with perspective containing
absolutely-centered `.card3d` elements:

- **Card**: `width: clamp(230px, 30vw, 320px); aspect-ratio: 3/4`, rounded, a big
  watermark letter, gradient art background, glass caption bar (title + category chip).
- **Data**: a `PROJECTS` array of `{t: title, c: categoryLabel, f: filterKey, g: cssGradient, l: bigLetter}`.
  Cards are built from it in JS; filters rebuild the list with a stage fade.
- **`layout()` math** (called on every state change; `off = i - (active + dragOffset)`,
  wrapped to shortest distance for a ring feel: `if(off > n/2) off -= n`):
  - `x = off * spacing()` (spacing ≈ 55–60% of card width, responsive)
  - `z = -min(|off|,4) * 150` · `ry = -off * 16deg` · scale ~`1 - abs*.08` (floor .78)
  - `opacity = 1 - abs*.18`, hidden ≥4 away; `zIndex = 100 - abs*10`
  - non-center cards: `filter: brightness(1-abs*.18) saturate(.85)`
  - center card: `is-active` class, big shadow + ember glow
- **Drag**: pointerdown/move/up on the stage. While held: add `.dragging` class
  (kills transitions → 1:1 follow), `dragOffset = -(dx)/spacing()` (fractional).
  Track `velocity = dx/dt`; on release snap to `Math.round(active+dragOffset)`,
  and if `|velocity| > .35` carry one extra card in the flick direction.
  A >6px move sets `didDrag` so click-through is suppressed.
- **Autoplay**: every 3.2s advance one card — ONLY while the section is on screen
  (IntersectionObserver) and not `reduce`; any interaction pauses it, resuming
  after 6s idle.
- **Scroll drift**: GSAP scrub tween on the whole stage, `x: 70 → −70` across the
  section's scroll span.
- **Mouse tilt**: on the active card only, fine pointers only, skipped while dragging:
  `rotateY(px*10) rotateX(-py*8) translateZ(40px) scale(1.02)`.
- **Mobile**: `touch-action: pan-y` on the stage so vertical page scroll still works.
- Prev/next round buttons (52px circles) + "01 / 08" counter; hint line
  "Drag, scroll the cards, or use the arrows".

### 4.6 Experience timeline
Vertical timeline of roles: a hairline track on the left with a gradient progress
line (`#expProgress`) that `scaleY` 0→1 scrubbed to scroll through the section.
Each entry: 11px dot on the line, dates (Space Grotesk label style), role + company
heading, one-line description, and a row of small brand chips. Entries reveal
(y + opacity) with stagger as they enter.

### 4.7 Contact
Full-width closer: masked 2-line headline ("Got a brand that / needs a *wow?*" —
same masked-line pattern as the hero), a huge soft radial glow orb behind,
then magnetic CTAs: primary email button, plus Behance / LinkedIn / phone links.

### 4.8 Footer
One hairline-topped row: © line left, location/tagline right, 12px muted text.

---

## 5. Responsive rules

- Floor: **360px**. Test at 360 / 768 / 980 / 1280 / 1600.
- Below **980px**: portrait dims and slides mostly off-edge (decorative), hero text
  goes full-width; carousel spacing tightens; nav links may collapse.
- ≥ **981px**: hero text column width-capped so it never underlaps the portrait.
- Wide content never causes horizontal page scroll.

## 6. Quality bar (ship checklist)

- [ ] Zero console errors; all CDN URLs return 200 (check the Network tab — the Lenis
      guard means a 404 fails *silently*)
- [ ] 60fps scroll — only `transform`/`opacity` animate in scroll/loop motion
- [ ] `prefers-reduced-motion` kills all decorative motion, content still readable
- [ ] Touch devices: no custom cursor, no magnetic pull, carousel drag works,
      vertical scroll never hijacked
- [ ] No FOUC (initial `gsap.set` before entrance timeline)
- [ ] No `localStorage`/`sessionStorage` usage
- [ ] After JS edits: `node --check` on the extracted script block

---

## 7. Content slots — what YOU replace

Everything personal lives in these slots; the rest of the file is reusable as-is.

| Slot | Where | Replace with |
|---|---|---|
| Name + role | `<title>`, meta description, logo, hero | your name, your discipline |
| Hero headline | hero `.line` spans | your 3-line hook — keep one italic gradient word |
| Portrait | base64 `#portraitImg` | your photo, B&W cut-out, ~800px wide, on transparency |
| Availability badge | hero | your status line |
| Skill tags #01–#04 | hero bottom | your 4 core skills |
| Marquee brands | marquee list (JS duplicates it) | brands/clients/tech you've worked with |
| About copy | about section | your bio, 2–3 short paragraphs |
| Stats | count-up numbers | your 3 metrics (years / projects / clients…) |
| Skill bars | about card | 4–5 skills with honest percentages |
| `PROJECTS` array | `{t, c, f, g, l}` objects in JS | your work: title, category label, filter key, card gradient, initial letter |
| Filter pills | work section | your 3 category keys (must match `f` values) |
| Experience entries | timeline | your roles: dates, company, description, chips |
| Contact | contact + footer + nav CTA | your email, phone, social links, city |
| Accent palette | `:root` tokens (optional) | your colors, per the re-theming rule in §2 |

## 8. Deploy

1. New GitHub repo → add `index.html` (+ this file) → push `main`.
2. Vercel → Import the repo → framework preset "Other", no build command,
   output dir = root. Every push to `main` auto-deploys.
3. If Vercel says the project name exists, give the *project* a new name during
   import — the repo name doesn't matter.

## 9. Suggested prompt for Claude Code

> Read TEMPLATE.md and the reference index.html. Build me the same single-file
> portfolio with my content: [name], [role], [city]. Hero line: "…". Projects: […].
> Experience: […]. Contact: […]. Keep every mechanic, timing, and quality rule
> from the template; change only the content slots and (optionally) the palette.
